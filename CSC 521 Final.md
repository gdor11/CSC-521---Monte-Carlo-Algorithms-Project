

```python
import csv
import math
from nlib import *
import random
from math import exp, log
import datetime
```


```python
import os
os.getcwd()
```




    'C:\\Users\\guy.dor\\Documents\\CSC 521'




```python
filename = 'accidents.csv'
history = []
log_loss = []
Ahist=[]
Bhist=[]
Aplant=[]
Aday=[]
Ayear=[]
Aloss=[]
Alog_loss=[]
Bplant=[]
Bday=[]
Byear=[]
Bloss=[]
Blog_loss=[]

with open(filename) as myfile:
    reader = csv.reader(myfile)
    header = ['Plant','Day','Loss']
    rows = [dict(zip(header,row)) for row in reader]
    rows.reverse()
    for k,row in enumerate(rows):
        plant = row['Plant']
        day = int(row['Day'])      
        loss = float(row['Loss'])
        log_loss = math.log(loss)
        if 0<day<=365:
            year=1
        elif 366<day<=730:
            year=2
        elif 730<day<=1095:
            year=3
        elif day>1095:
            year=4
        
        history.append([plant, day, year, loss, log_loss])
        if plant=='A':
            Ahist.append([plant, day, loss, log_loss])
            Aplant.append(plant)
            Aday.append(day)
            Ayear.append(year)
            Aloss.append(loss)
            Alog_loss.append(log_loss)
        else: 
            Bhist.append([plant, day, loss, log_loss])
            Bplant.append(plant)
            Bday.append(day)
            Byear.append(year)
            Bloss.append(loss)
            Blog_loss.append(log_loss)
            
            
        print plant, day, year, loss, log_loss
    
```

    B 1462 4 3626.0 8.19588539131
    A 1455 4 11561.0 9.35539264368
    A 1453 4 34881.0 10.4596975473
    B 1452 4 1718.0 7.44891610254
    B 1449 4 1904.0 7.55171221535
    A 1443 4 9923.0 9.20261057391
    B 1441 4 1218.0 7.10496544827
    A 1440 4 4400.0 8.38935981991
    A 1430 4 2767.0 7.92551897979
………………………………………………………………………………………
    


```python
#Average number of acidents/year in plant A
def accidents_in_year(n,plant):
    accidents=0
    for i in plant:
        if i==n:
            accidents+=1
    return accidents
(accidents_in_year(1, Ayear)
+accidents_in_year(2, Ayear)
+accidents_in_year(3, Ayear)
+accidents_in_year(4, Ayear))/4


```




    33




```python
#Average number of acidents/year in plant B
(accidents_in_year(1, Byear)
+accidents_in_year(2, Byear)
+accidents_in_year(3, Byear)
+accidents_in_year(4, Byear))/4
```




    39




```python
#Average loss per accident in plant A

def Expected(val):
    r=float(sum(val))/len(val)
    return r
Expected(Aloss)
```




    17470.155555555557




```python
#Average loss per accident in plant B
Expected(Bloss)

```




    2022.9615384615386




```python
#Average Total loss per year in plant A
Expected(Aloss)*len(Ahist)/4

```




    589617.75




```python
#Average Total loss per year in plant B
Expected(Bloss)*len(Bhist)/4
```




    78895.5




```python
#Average Log loss per accident in plant A
Expected(Alog_loss)
```




    9.146099829385644




```python
#Average Log loss per accident in plant B
Expected(Blog_loss)
```




    6.95909167211235




```python
#Standard Deviation of Log loss per accident in plant A
sd(Alog_loss)
```




    1.061510583198764




```python
#Standard Deviation of Log loss per accident in plant A
sd(Blog_loss)
```




    1.133381024878168




```python
#Simulate once for one year of losses for plants A and B

def simulate_once():
        N = 1
        max_time = 365 # 1 year
        xm_a = 9.146099829385644  # mu of log loss A
        sd_a = 1.061510583198764  # sd of log loss A
        lamb_a = float(33)/365 # accidents per day

        xm_b = 6.95909167211235  # mu of log loss B
        sd_b = 1.133381024878168  # sd of log loss B
        lamb_b = float(39)/365 # accidents per day

        time = [0.0] * N

        ta = 0  # timer for loss at plant A
        tb = 0  # timer for loss at plant B
        sima=0  # Loss aggregator for plant A
        va=[]
        simb=0  # Loss aggregator for plant A
        vb=[]
        hista=[]
        histb=[]
        
        while True:
            ta = ta + random.expovariate(lamb_a) # time between losses~exponential dist for plant A with lambda=lamb_a
            if ta > max_time: break
            
            for k in range(N):
                if time[k] <= ta:
                    rva=(exp(random.gauss(xm_a,sd_a))) # Log Loss for A~N(xm_a,sd_a). exp() transforms log of Loss into Loss
                    sima+=rva
                    va.append(rva)
                    hista.append(sima)
                else:
                    sima+=(0)
                    va.append(0)
                    hista.append(0)

        while True:
            tb = tb + random.expovariate(lamb_b) # time between losses~exponential dist for plant B with lambda=lamb_b
            if tb > max_time: break
            for k in range(N):
                if time[k] <= tb:
                    rvb=(exp(random.gauss(xm_b,sd_b))) # Log Loss for B~N(xm_b,sd_b). exp() transforms log of Loss into Loss
                    simb+=rvb
                    vb.append(rvb)
                    histb.append(simb)
                else:
                    sima+=(0)
                    va.append(0)
                    hista.append(0)
        total_hist=[]
        th=list(map(sum,zip(hista,histb)))
        for i in th:
            total_hist.append(((th.index(i)),(i)))
        
        Canvas().plot(total_hist).save('Loss_Timeplot.png')
        Canvas().hist(va).save('simulated_loss_Plant_A.png')
        Canvas().hist(vb).save('simulated_loss_Plant_B.png')

        return (sima+simb)




simulate_once()

```




    665118.599115779




```python
#Simulate many for one year of losses for plants A and B

class LossSimulator(MCEngine):
    def __init__(self, N):
        self.N = N
    def simulate_once(self):
        N = self.N
        max_time = 365 # 1 year
        xm_a = 9.146099829385644  # mu of log loss A
        sd_a = 1.061510583198764  # sd of log loss A
        lamb_a = float(33)/365 # accidents per day
        
        xm_b = 6.95909167211235  # mu of log loss B
        sd_b = 1.133381024878168  # sd of log loss B
        lamb_b = float(39)/365 # accidents per day

        time = [0.0] * N

        ta = 0  # timer for loss at plant A
        tb = 0  # timer for loss at plant B
        sima=0  # Loss aggregator for plant A
        simb=0  # Loss aggregator for plant A
        
        while True:
            ta = ta + random.expovariate(lamb_a) # time between losses~exponential dist for plant A with lambda=lamb_a
            if ta > max_time: break
            for k in range(N):
                if time[k] <= ta:
                    sima+=(exp(random.gauss(xm_a,sd_a))) # Log Loss for A~N(xm_a,sd_a). exp() transforms log of Loss into Loss
                else:
                    sima+=(0)

        while True:
            tb = tb + random.expovariate(lamb_b) # time between losses~exponential dist for plant B with lambda=lamb_b
            if tb > max_time: break
            for k in range(N):
                if time[k] <= tb:
                    simb+=(exp(random.gauss(xm_b,sd_b)))  # Log Loss for B~N(xm_b,sd_b). exp() transforms log of Loss into Loss
                else:
                    simb+=(0)

        return (sima+simb)/N

sim=LossSimulator(5000)
print sim.simulate_many(rp=.1)
```

    (588034.2067844407, 635101.8950825462, 686214.3608994514)
    


```python
# find 90% coverage of loss
v = []

for i in range(100):
    v.append(sim.simulate_once())
v.sort()
print v[90]
```

    773442.953903
    

