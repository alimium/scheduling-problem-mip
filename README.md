## Further work
Below are future plans and further ideas:
- Implementation of the problem using genetic algorithms.
- Fine-tuning solver hyperparameters to achieve best results.


---
## Introduction & Problem Definition
Suppose an educational institute has certain students each of whom are enrolled in certain courses. A final exam is going to be held for each course. All exams must take place within a specific time window. _(i.e. one weeks)_ During these dates, there are specific valid time periods that exams can be held at. _(i.e. 8 AM - 10 AM, 10 AM - 12 PM, 14 PM - 16 PM)_ Also, all exams must be held in available rooms which have certain capacities.
> **Notice**  
> The list of students is available. In other words it is known that which student is enrolled in what courses.

**Mandatory Constraints:**
1. Each course's exam must be held on a specific day and time period.
2. Each student must have at most one exam at a single time period on a day.
3. Each course's exam must be held in at least one room.
4. Room capacities must be considered.
5. The number of used rooms at a single time period must not exceed the total number of available rooms.

**Preferred constraints: (In order of preference)**
1. Each student should preferably have no more than one exam in a day.
2. Each student should preferably have at least one day off between their exams.
3. Preferably, only one course exam should be scheduled in each room during a given time period.

---
## Model Summary
A brief summary of the model is presented below. This can be used to set up a solver environment for this problem. For detailed model description please read the documentation.

### Sets:
|Definition|Symbol|Description|
|:-----------|:--------:|:-------------|
|$\mathbb{C} = \set{\dots}$|$c,c^\prime$|The set of courses.|
|$\mathbb{D} = \set{\dots}$|$d,d^\prime$|The set of valid days to hold the exams.|
|$\mathbb{H} = \set{\dots}$|$h,h^\prime$|The set of valid time periods in a day.|
|$\mathbb{R} = \set{\dots}$|$r$|The set of available rooms.|
|$\mathbb{S} = \set{\dots}$|$s$|The set of enrolled students.|

### Parameters:
$Cap_r$ : Maximum capacity for room $r$
$a_{s,c}=\begin{cases}1 & Student\;s\;is\;enrolled\;in\;course\;c \\ 0 & o.w. \end{cases}$  : Determines if student $s$ is enrolled in course $c$

### Variables:
#### Primary Variables:
Binary: $\delta{c,d,h,r}= 1$ If exam for course $c$ is held on day $d$ at time $h$ in room $r$.  
Non-Negative Integer: $x_{c,d,h,r}$ : Number of enrolled students in course $c$ whose exam is held in room $r$.

#### Intermediate Variables:
> **Attention**  
> Non Negative Reals are used instead of Non Negative Integers for intermediate variables to enhance solver performance.
1. Binary: $\psi_{s,c,d,h}$ (In 2<sup>nd</sup> hard constraint)
2. Binary: $\gamma_{s,d,h}$ (In 1<sup>st</sup> soft constraint)
3. Non-Negative Real: $\lambda_{s,d,h}$ (In 1<sup>st</sup> soft constraint)
4. Non-Negative Real: $\lambda_{s,d,h}^\prime$ (In 1<sup>st</sup> soft constraint)
5. Binary: $\sigma_{s,d,d^\prime}$ (In 2<sup>nd</sup> soft constraint)
6. Non-Negative Real: $\beta_{s,d,d^\prime}$ (In 2<sup>nd</sup> soft constraint)
7. Non-Negative Real: $\beta_{s,d,d^\prime}^\prime$ (In 2<sup>nd</sup> soft constraint)
8. Non-Negative Real: $\zeta_{d,h,r}$ (In 3<sup>rd</sup> soft constraint)

#### Relations:
$$\delta_{c,d,h,r} \geq 1 \implies x_{c,d,h,r} \geq 1 \hspace{1cm} \forall c,d,h,r$$
$$\delta_{c,d,h,r} \leq 0 \implies x_{c,d,h,r} \leq 0 \hspace{1cm} \forall c,d,h,r$$
$$\sum_{d,h,r}x_{c,d,h,r} = \sum_{s} a_{s,c} \hspace{1cm} \forall c \hspace{0.3cm}$$

### Objective Function:
$$\begin{equation}\begin{aligned} min \;\; z & = \sum_{s,d,h}U_{s,d,h}\cdot\lambda_{s,d,h}
 +\sum_{s,d,h}U_{s,d,h}^\prime\cdot\lambda_{s,d,h}^\prime \\\\ &
 +\sum_{s,d,d^\prime}V_{s,d,d^\prime}\cdot\beta_{s,d,d^\prime} +\sum_{s,d,d^\prime}V_{s,d,d^\prime}^\prime\cdot\beta_{s,d,d^\prime}^\prime \\\\ &
 +\sum_{d,h,r}W_{d,h,r}\cdot\zeta_{d,h,r}
 \end{aligned}\end{equation}$$

### Constraints:
$$\sum_{d,h,r} \delta_{c,d,h,r} \geq 1 \hspace{1cm} \forall c$$
$$\sum_{d^\prime, h^\prime, r^\prime : (d,h)\neq(d^\prime, h^\prime)} \delta_{c,d^\prime,h^\prime, r^\prime} \leq |\mathbb{D \times H \times R}|\cdot(1-\delta_{c,d,h,r})\hspace{1cm} \forall s,c,d,h,r$$
$$\sum_{r}a_{s,c}\cdot\delta_{c,d,h,r} \leq \|\mathbb{R}\|\cdot \psi_{s,c,d,h} \hspace{1cm} \forall s,c,d,h$$
$$\sum_{c^\prime,r \; \colon \; c \neq c^\prime} a_{s,c^\prime}\cdot\delta_{c^\prime,d,h,r} \leq\|\mathbb{C\times R}\|\cdot (1-\psi_{s,c,d,h}) \hspace{1cm} \forall s,c,d,h$$
$$\sum_{c}x_{c,d,h,r} \leq Cap_r \hspace{1cm} \forall d,h,r$$
&nbsp;
$$\sum_{r, c} a_{s,c}\cdot \delta_{c,d,h,r} - \lambda_{s,d,h} \leq  \|\mathbb{R\times C}\|\cdot \gamma_{s,d,h} \hspace{1cm} \forall s,d,h$$
$$\sum_{r, c, h^\prime \neq h} a_{s,c}\cdot \delta_{c,d,h^\prime,r} - \lambda_{s,d,h}^\prime\leq \|\mathbb{C\times R \times H\|}\cdot (1-\gamma_{s,d,h}) \hspace{1cm} \forall s,d,h$$
$$\sum_{c,h,r} a_{s,c}\cdot \delta_{c,d,h,r} - \beta_{s,d,d^\prime} \leq 0 + \|\mathbb{C\times R \times H\|}\cdot \sigma_{s,d,d^\prime}\hspace{1cm} \forall s,d,d^\prime$$
$$\sum_{c, h,r} a_{s,c}\cdot \delta_{c,d^\prime,h,r} -\beta_{s,d,d^\prime}^\prime \leq 0 + \|\mathbb{C\times R \times H\|}\cdot (1-\sigma_{s,d,d^\prime}) \hspace{1cm} \forall s,d,d^\prime$$
$$\sum_{c} \delta_{c,d,h,r} - \zeta_{d,h,r}\leq 1 \hspace{1cm}\forall d,h,r$$
> **Note:**  
> Extras For Modeling Accuracy
> #### Intermediate Variables:
> Binary: $\omega_{s,d,h}$
> Binary: $\omega_{s,d,h}^\prime$
> #### Constraints:
> $$\lambda_{s,d,h}\leq\|\mathbb{R\times C}\|\cdot\omega_{s,d,h}\hspace{1cm}\forall s,d,h$$
> $$\sum_{h^\prime \neq h}\lambda_{s,d,h^\prime} \leq \|\mathbb{H}\|\cdot\|\mathbb{R\times C}\|\cdot(1-\omega_{s,d,h}) \hspace{1cm}\forall s,d,h$$
> $$\lambda_{s,d,h}^\prime\leq\|\mathbb{R\times C}\|\cdot\omega_{s,d,h}^\prime \hspace{1cm}\forall s,d,h$$
> $$\sum_{h^\prime \neq h}\lambda_{s,d,h^\prime}^\prime \leq \|\mathbb{H}\|\cdot\|\mathbb{R\times C}\|\cdot(1-\omega_{s,d,h}^\prime) \hspace{1cm}\forall s,d,h$$


---
### Results
A sample dataset was generated to ensure all constraints (hard and soft) are being checked as well as the feasibility of the model. The data is described below.

#### Students

There are 4 students and 7 courses.

|Course|Student|
|------|-------|
|$c_1$ |$s_1$  |
|$c_2$ |$s_2$, $s_3$|
|$c_3$ |$s_1$, $s_2$, $s_3$|
|$c_4$ |$s_4$  |
|$c_5$ |$s_2$  |
|$c_6$ |$s_3$  |
|$c_7$ |$s_2$, $s_3$, $s_4$|

#### Days, Time Periods, Rooms
$D = \set{1,2}$
$H = \set{1,2}$  ( 1:(8-10 AM),  2:(10-12 AM) )
$R = \set{1,2}$
$Cap_{r_1} = 1, Cap_{r_2} = 2$

#### Outputs
Initially, the model was solved on the above sample dataset without the consideration of constraint 1 extras. GLPK Integer optimizer 5.0 solver was used and the time taken was 10.2 seconds. The model outputs are as follows:

|             |Room 1 (cap=1)|Room 2(cap=2)|
|-------------|------|------|
|$d_1$ : $h_1$|$c_5$ [$s_2$]|$c_1$ [$s_1$], $c_6$ [$s_3$]|
|$d_1$ : $h_2$|$c_7$ [$s_2$]|$c_7$ [$s_3$, $s_4$]|
|$d_2$ : $h_1$|$c_3$ [$s_1$]|$c_3$ [$s_2$, $s_3$]|
|$d_2$ : $h_2$|$c_4$ [$s_4$]|$c_2$ [$s_2$, $s_3$]|

As it can be seen, no student has more than one exam in a day, students $s_2, s_3$ have consecutive exams and room 2 holds 2 course's exams at the same time on day 1 and time period 1.

Then we included the constraint extras for the first hard constraint and the model performance improved significantly on the same dataset. time taken reduced to 3.8 seconds and with the search tree being remarkably smaller, memory usage improved by a factor of 3.7. The solver produced the following results.

|             |Room 1 (cap=1)|Room 2(cap=2)|
|-------------|------|------|
|$d_1$ : $h_1$|$c_4$ [$s_4$]|$c_2$ [$s_2$, $s_3$]|
|$d_1$ : $h_2$|$c_3$ [$s_1$]|$c_3$ [$s_2$, $s_3$]|
|$d_2$ : $h_1$|$c_7$ [$s_2$]|$c_7$ [$s_3$, $s_4$]|
|$d_2$ : $h_2$|$c_5$ [$s_2$]|$c_1$ [$s_1$],$c_6$ [$s_3$]|

This result is valid as well with minimal violations of the second and third soft constraints. Students $s_2, s_3$ have consecutive exams and room 2 holds exams for courses $c_1, c_6$ on day 2 at the second time period.

Moreover, we have tried to solve the model on a larger dataset with 12 courses, 8 students, 3 days, 2 time periods and 2 rooms and by the 100 minute mark, we achieved 85% MIP gap within the 100 minute mark. However, due to the GLPK solver being a single core process, we were not able to complete a full solution.
