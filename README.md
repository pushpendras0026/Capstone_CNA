# 📘 Dynamic Pricing for Urban Parking Lots - Capstone Project (Summer Analytics 2025)

This repository contains the implementation of three dynamic pricing models for urban parking lots using real-time data streams. The project was developed using only `Python`, `Pandas`, `Numpy`, and `Pathway`, as per the challenge guidelines.

---

## 📦 Libraries Used and Their Purpose

| Library   | Purpose                                                                  |
| --------- | ------------------------------------------------------------------------ |
| `pandas`  | Data manipulation, preprocessing, and aggregation                        |
| `numpy`   | Mathematical operations and array manipulation                           |
| `pathway` | Real-time data streaming, windowing, and stateful computation            |
| `math`    | Custom mathematical transformations (e.g., smoothing, haversine formula) |

---

## ✅ Project Overview

Urban parking lots face imbalances in demand throughout the day. This project builds a dynamic pricing engine that adjusts parking prices in real-time based on live features like occupancy, queue, traffic, special day flags, vehicle type, and competition.

### Models Implemented:

1. **Model 1: Baseline Linear Model (Time-local demand)**
2. **Model 2: Demand-Based Dynamic Model (Global demand)**
3. **Model 3 (Optional): Competitive Pricing Model using Geographic Proximity**

Each model is executed in a real-time streaming setup using Pathway with individual streams for each parking space.

---

## 🧠 Model 1: Baseline Linear Demand Model

### ➤ Demand Function:

```python
Demand_t = alpha * Occupancy_ratio + beta * QueueLength_pressure
```

Where:

* `Occupancy_ratio = Occupancy / Capacity`
* `QueueLength_pressure = QueueLength / Capacity`

### ➤ Normalization:

Demand is normalized with respect to the **first reading** of the day for each lot.

```python
normalized_demand = demand / demand_at_8AM
```

### ➤ Smoothing Function:

Smooths the demand to avoid erratic pricing changes:

```python
def custom_smooth_demand(x, p=1.5, q=1.5):
    if x < 1:
        return -0.5 * (1 - x)**p
    else:
        return 1 - 1 / ((x + 1)**q)
```

### ➤ Price Formula:

```python
def smooth(demand, base=10):
    lam = custom_smooth_demand(demand)
    return base * (1 + lam)
```

### ➤ Window:

* Tumbling window of **2 hours** for each `SystemCodeNumber`
* Price recalculated separately for each lot in real-time

---

## 🧠 Model 2: Feature-Rich Demand-Based Dynamic Model

### ➤ Demand Function:

```python
Demand_t = (
    alpha * max_Occupancy_ratio +
    beta * max_QueueLength_pressure +
    gamma * max_Traffic +
    delta * max_IsSpecialDay +
    epsilon * max_VehicleType
)
```

* Uses **maximum observed values** in the window for each feature.
* This allows the model to account for peak demand conditions.

### ➤ Normalization:

```python
normalized_demand = demand / max(demand_all_lots)
```

* Demand is normalized globally using the **maximum demand observed** across the entire dataset.

### ➤ Price Update:

```python
Price_t = base_price * (1 + lambda * normalized_demand)
```

* Ensures that price changes are smooth and never exceed 2x or drop below 0.5x of base.

### ➤ Streaming:

* Run separately for **each parking space**
* Real-time, windowed demand calculations using `Pathway`

---

## 🧠 Model 3 (Optional): Competitive Pricing Model

### ➤ Purpose:

Incorporate geographic competition to intelligently adjust price:

1. If lot is **full** and **nearby lots are cheaper**: lower price or reroute
2. If **nearby lots are expensive** and your lot is underused: raise price

### ➤ Steps:

1. **Join lots within 500m** using the Haversine formula:

```python
def haversine_km(lat1, lon1, lat2, lon2):
    ...
```

2. **Self-join** on lat-long in Pathway using `pw.apply()`
3. **Aggregate competitor stats**:

```python
avg_competitor_price = mean(price of nearby lots)
```

4. **Adjustment Rules**:

```python
if occ_ratio >= 0.9 and min_competitor_price < current_price:
    new_price = max(current_price - 1, 7)
elif occ_ratio <= 0.5 and avg_competitor_price > current_price:
    new_price = min(current_price + 1.5, 20)
```

---

## 🧪 Real-Time Setup with Pathway

### ➤ Key Pathway Features Used:

* `pw.temporal.tumbling()`: Windowing logic
* `pw.apply()`: Custom logic in stream
* `pw.reducers`: Aggregations (mean, max, first, etc.)
* `pw.demo.replay_csv()`: Stream CSV in real-time
* `pw.io.print()`: Output live price stream

---

## 📊 Visualization Plan (Planned / Optional)

* Real-time pricing curves using **Bokeh**
* Overlay competitor prices
* Map-based visualization using **Folium** or **Plotly**

---

## 📎 References

* [Pathway Docs - First App](https://pathway.com/developers/user-guide/introduction/first_realtime_app_with_pathway/)
* [Pathway Docs - From Jupyter to Deploy](https://pathway.com/developers/user-guide/deployment/from-jupyter-to-deploy/)
* [Summer Analytics 2025](https://www.caciitg.com/sa/course25/)

---

## 👨‍💻 Author

**Pushpendra Singh**
Summer Analytics 2025 Participant
Contact: (pushpendras026@iitg.ac.in)
