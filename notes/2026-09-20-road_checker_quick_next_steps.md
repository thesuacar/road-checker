# Road Checker – Quick Next Steps

## Current status

The first annotated recording already shows a **clear preliminary difference between smooth and bumpy road segments**.

Using 2 s windows on the vertical dynamic acceleration:

- **Bumpy RMS:** ~2.13 m/s²
- **Smooth RMS:** ~0.82 m/s²

The frequency-domain view also shows noticeably more vibration energy in the bumpy sections.

> This is only a preliminary check because the absolute annotation offset still needs to be confirmed, but the signal already looks very feasible to separate.


Check this initial image (we should not use this in the report)

![[road_checker_preliminary_bumpy_vs_smooth.png]]

---

## Practical plan

### 1. Fix the annotation alignment
Treat annotations as **state-change markers**:

- `bumpy starts` → everything after this is bumpy until the next marker
- `smooth starts` → everything after this is smooth until the next marker

Ignore marker press duration.

Confirm the timestamp of the first marker relative to the sensor recording, then propagate all remaining marker times from there.

Add a small guard band (~1–2 s) around transitions.

---

### 2. Start with physically meaningful signals

Use:

- **Accelerometer** → main road-vibration signal
- **Gravity** → estimate phone orientation / vertical direction
- **Gyroscope** → additional rotational/vibration information

Useful derived signals:

- vertical dynamic acceleration
- acceleration magnitude
- gyroscope magnitude
- jerk / rate of change of acceleration

Do not start with PCA or a complex model yet.

---

### 3. Window the signal

Start simple:

- **2 s windows**
- **50% overlap**
- discard windows crossing a smooth/bumpy transition

Each window becomes one sample for later analysis/modeling.

---

### 4. Extract a small set of interpretable features

Start with:

- RMS
- standard deviation
- peak-to-peak
- jerk RMS
- gyroscope RMS
- energy / power in a few frequency bands

Example frequency bands to investigate:

- 0.5–3 Hz
- 3–10 Hz
- 10–20 Hz
- 20–40 Hz

Do not assume these are final; use the data to confirm where separation actually occurs.

---

### 5. Compare smooth vs bumpy before doing ML

For each feature:

- plot distributions / boxplots
- compare mean and median
- inspect overlap between classes
- inspect representative time signals
- inspect PSD / frequency content

Goal:

> Identify which features physically separate smooth and bumpy road conditions.

---

### 6. Feature selection / dimensionality reduction

After the physical features exist:

- use **Mutual Information** to identify features related to the label
- use correlation to identify redundant features
- use **PCA** mainly to inspect redundancy / structure or reduce dimensions if useful

Remember:

> PCA finds variance, not necessarily class relevance.

---

### 7. Build a simple baseline first

Before complex models, test whether a very simple rule or classifier already separates the classes well.

Then implement the supervised / unsupervised models required by the assignment.

If a simple model performs similarly to a complex one, that is a useful engineering result.

---

### 8. Avoid data leakage

**Do not randomly split overlapping windows from the same recording between train and test.**

Prefer splitting by:

- complete recording
- participant
- location / road section

This tests whether the model learned **road quality**, rather than memorizing one recording/person/device.

---

### 9. Collect more data only when the analysis tells us what is missing

Do not collect hours of data just for volume.

Prioritize diversity:

- different participants
- different road locations
- different kinds of smooth/bumpy surface
- reasonable variation in cycling speed

Start with the planned recordings, analyze them, then collect additional data only if specific failure cases appear.

---

## Short version

**Raw sensors → align annotations → derive physical signals → window → extract simple features → compare smooth/bumpy → select features → simple baseline → required models → validate on independent recordings.**

The preliminary recording already suggests that the road conditions are separable. The next step is to quantify that difference systematically rather than jumping directly into complex ML.


---

## Appendix A – Operational-state logic before road classification

### Why this layer is needed

Before classifying the road as **smooth** or **bumpy**, the system should first verify whether the sensor data comes from a **valid riding state**.

This prevents the road classifier from being confused by events such as:

- placing the phone in the pocket;

- taking the phone out;

- standing still at a traffic light;

- initial push-off / starting to move;

- braking to a stop;

- other non-steady transitions.


The key idea is:

> The final road classifier should remain a **2-class model** (`smooth` vs `bumpy`), but it should only run when the bicycle is already in a suitable **“in movement”** operating state.

---

> In my head, we should only classify while **`in movement`**

![[Pasted image 20260920175641.png]]

### Global logic

A practical high-level flow is:

1. **Start**

2. **Time-domain state verification**

3. Decide whether the current signal window corresponds to:

- `stopped`
	
- `stopping`
	
- `starting to move`
	
- `in movement`
	
1. Only when the system is confidently in the `**in movement**` state, trigger the road classifier.

2. Run the road classifier in a **moving window**, simulating real time.

3. Output:

- `Class 0 = smooth`
	
- `Class 1 = bumpy`
	
1. Keep monitoring the state in parallel.

2. If the system leaves the `in movement` state, stop terrain classification and return to state verification.


---

### Recommended interpretation of the sensors

The sensor channels should first be interpreted in terms of **physical meaning**, not just raw `x`, `y`, `z` values.

Useful derived channels:

- **Dynamic acceleration magnitude**
    
- **Acceleration along gravity**
    
- **Acceleration perpendicular to gravity**
    
- **Gyroscope magnitude**
    
- **Jerk** (rate of change of acceleration)
    

Using the **gravity vector** is especially useful because it reduces dependence on the exact phone orientation inside the pocket. Even if the phone rotates slightly, the “vertical” direction can still be estimated.

---

### Separation of responsibilities

#### Layer 1 – Operational-state verification

Purpose: decide whether the data window is valid for road classification.

This layer should detect conditions such as:

- phone handling;
    
- stopped bicycle;
    
- start-up transient;
    
- stopping transient;
    
- unstable transition periods.
    

This layer is mainly a **gating / validation** step.

#### Layer 2 – Road classifier

Purpose: when the system is already in a valid riding state, classify the road as:

- `smooth`
    
- `bumpy`
    

This layer should focus on **road roughness**, not on whether the bike is moving or not.

---

### Intuition for state detection

A good first engineering assumption is:

- **Stopped**  
    Very low vibration energy; little sustained motion.
    
- **Starting to move**  
    A transient interval with changing acceleration patterns, often stronger low-frequency body/bike motion.
    
- **In movement**  
    Sustained motion with repeatable vibration content.  
    This is the valid state for road classification.
    
- **Stopping**  
    Another transient interval, where the signal becomes less representative of the road itself.
    

These states do not need to be perfect on day one. A simple heuristic gate is already valuable.

---

### Practical implementation idea

Use a short sliding window and evaluate simple time-domain features such as:

- RMS
    
- standard deviation
    
- peak-to-peak
    
- jerk RMS
    
- gyroscope RMS
    

Then apply simple rules or a lightweight classifier to determine whether the current window is:

- invalid / not suitable for road classification, or
    
- valid / suitable for road classification.
    

Only then pass the window to the `smooth` vs `bumpy` classifier.

---

### Important modeling principle

Do **not** force all behaviors into the terrain classifier.

For example:

- placing phone in pocket ≠ smooth road
    
- stopping at a light ≠ smooth road
    
- starting to pedal ≠ bumpy road
    

These are **operational states**, not terrain classes.

By separating these concerns, the system becomes easier to understand, easier to debug, and more physically meaningful.

---

### Suggested state-machine view

A simple conceptual state machine is:

- `Start`
    
- `State verification`
    
    - if `stopped` → stay in verification
        
    - if `starting to move` → stay in verification
        
    - if `in movement` → trigger classifier
        
- `Road classification`
    
    - output `smooth` or `bumpy`
        
    - keep checking whether motion is still valid
        
    - if motion ends or becomes unstable → return to verification
        

This is a very good fit for a **moving-window, near-real-time pipeline**.

---

### Short takeaway

> First determine whether the system is in a valid riding state.  
> Only then classify the terrain.

This keeps the project physically grounded and should make the later feature engineering and classification work much more robust.