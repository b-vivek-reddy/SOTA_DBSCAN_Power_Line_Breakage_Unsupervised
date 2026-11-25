# DBSCAN-Based Localization of Power Line Breakage Zones Using Voltage and Current Sensor Anomaly Clustering

An unsupervised fault-localization pipeline that detects broken conductors and high-impedance faults (HIFs) on power distribution feeders. By computing robust anomaly scores from time-synchronized sensor readings and clustering co-occurring events across feeder network distance and time, this system isolates likely line-breakage zones and validates them against maintenance-crew fault reports.

---

## 📌 Overview & Motivation

Broken or downed conductors and high-impedance faults (HIFs) draw too little current to trip conventional overcurrent protection relays (only ~17% of HIFs are detected by standard relaying). Utilities often rely on customer calls and manual feeder patrols, creating severe wildfire and public safety risks. 

Unlike supervised ML approaches that require labeled fault datasets or single-point monitors that ignore spatial context, this project uses **DBSCAN** to aggregate co-occurring voltage and current anomalies across multiple sensors without needing a preset number of clusters.

### Key Objectives
* **O1:** Collect and synchronize voltage and current sensor readings across distribution feeder segments.
* **O2:** Apply spatial-temporal DBSCAN clustering to group co-occurring anomalous readings into likely breakage zones.
* **O3:** Validate predicted breakage zones against historical maintenance-crew fault-location reports.

---

## 🏗️ System Architecture

The pipeline is structured into three distinct layers:

```text
┌─────────────────────────────────────────────────────────────────────────┐
│ 1. DATA LAYER                                                           │
│    Sensors / RTUs (|V|, |I|, angles) ──▶ Time Sync ──▶ Feeder Topology  │
└────────────────────────────────────┬────────────────────────────────────┘
                                     ▼
┌─────────────────────────────────────────────────────────────────────────┐
│ 2. ANALYTICS LAYER                                                      │
│    Robust Z-Score & VUF/CUF ──▶ Event Construction ──▶ DBSCAN Clustering│
└────────────────────────────────────┬────────────────────────────────────┘
                                     ▼
┌─────────────────────────────────────────────────────────────────────────┐
│ 3. VALIDATION LAYER                                                     │
│    Zone Bounding ──▶ Match w/ Crew Reports ──▶ Hit Rate, F1 & Zone Map  │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## ⚙️ Methodology & Algorithms

### 1. Feature Extraction & Anomaly Scoring
For each sensor $s$ at time $t$, features include voltage magnitude ($|V|$), current magnitude ($|I|$), Voltage Unbalance Factor ($VUF$), and Current Unbalance Factor ($CUF$):

$$VUF = \frac{|V_2|}{|V_1|} \times 100\%, \quad CUF = \frac{|I_2|}{|I_1|} \times 100\%$$

Anomalies are scored using a **robust z-score** against a time-of-day baseline (median $m$ and median absolute deviation $MAD$ for time bin $h$):

$$z(s,t) = \frac{|x(s,t) - m(s,h)|}{1.4826 \cdot MAD(s,h)}, \quad \text{flag if } z \ge \tau$$

### 2. Event Construction & ST-DBSCAN Clustering
Flagged readings (excluding logged switching and planned outages) are converted into events $e_i = (d_i, t_i, a_i)$, representing feeder chainage $d_i$, timestamp $t_i$, and severity $a_i = \max z$. 

Events are clustered using both **network tree distance** ($\epsilon_s$) and **temporal proximity** ($\epsilon_t$):

$$N(i) = \{ j : d_{\text{net}}(i,j) \le \epsilon_s \text{ and } |t_i - t_j| \le \epsilon_t \}, \quad \text{core point if } |N(i)| \ge \text{minPts}$$

* Isolated alarms that do not meet `minPts` are discarded as noise.
* Spatial radius $\epsilon_s$ is selected via the knee of the $k$-distance plot.

### 3. Breakage Zone Mapping
Each valid cluster $C_k$ is mapped to a bounded feeder zone $Z_k$ with a confidence score equal to the sum of its event severities:

$$Z_k = \text{segments from nearest normal sensor upstream of } \min_{i \in C_k} d_i \text{ to nearest normal sensor downstream of } \max_{i \in C_k} d_i$$

---

## 📊 Evaluation Plan & Baselines

* **Metrics:** Hit rate, predicted zone length (km & segments), distance error, Precision/Recall/F1-score against crew reports, false alarms per feeder-month, and detection latency.
* **Baselines:** 
  1. Simple segment-bounding rule (last normal to first abnormal sensor)
  2. K-Means clustering
  3. HDBSCAN
  4. Classical impedance-based fault location (IEEE Std C37.114)
* **Testbeds:** Utility feeder sensor logs supplemented by **IEEE 13-node** and **IEEE 123-bus** simulated distribution feeders.

---

## 🗓️ Project Roadmap

| Phase | Target Period | Status |
| :--- | :--- | :--- |
| Literature Survey (SOTA Review) | Month 1 | ✅ Completed |
| Data Collection & Preprocessing (O1) | Month 2 | 🔄 Planned |
| Model Design: Anomaly Detection & DBSCAN (O2) | Month 3 | 📅 Planned |
| Experimentation & Crew-Report Validation (O3) | Month 4 | 📅 Planned |
| Final Report Writing & Submission | Month 5 | 📅 Planned |

---

## 👥 Team & Supervision

* **Author:** Bujunuri Vivek (Reg. No: `12408111`, Roll No: `14`)
* **Supervisor:** Dr. Bhupinder Singh
