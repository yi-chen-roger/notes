# MapReduce
## 1.Introduction to MapReduce
- What is MapReduce
- How MapReduce takes care of fault tolerant
- analytics can be performed by map reduce and the limitations

## 2. What is MapReduce
- Programming model?
- Execution environment?
- Software package?

Hadoop
- Distributed storage
- Distributed computation
- Fault tolerance


## 3. What is MapReduce
![](images/2020-08-11-15-34-03.png)

## 4. Learning Via Aggregation Statistics
![](images/2020-08-11-15-36-25.png)

## 5. MapReduce Abstraction
![](images/2020-08-11-16-00-25.png)
![](images/2020-08-11-16-09-20.png)
![](images/2020-08-11-16-09-55.png)


## 6. MapReduce System
![](images/2020-08-11-16-13-01.png)
![](images/2020-08-11-16-13-48.png)
Three stages
- Map Stage
- Shuffle Stage
- Reduce Stage

## 7. MapReduce Fault Recovery
recompute

## 8. Distributed File Systems
HDFS


## 9. MapReduce Design Choice
- What functionality can we remove?
![](images/2020-08-11-16-22-35.png)

## 10. Analytics with MapReduce
- classification/ KNN
- Regression

## 11. MapReduce KNN
![](images/2020-08-11-16-25-48.png)


## 12. Linear Regression
![](images/2020-08-11-16-28-01.png)
n - is the number of patients
d - number of feature,

![](images/2020-08-11-16-29-28.png)
- This can be easily done in small dataset
- will not work on big data
![](images/2020-08-11-16-30-58.png)

## 13. MapReduce for Linear Regression Quiz

![](images/2020-08-11-16-31-53.png)


## 14. Limitations of MapReduce
##### Why not logistic regression
![](images/2020-08-11-16-33-04.png)
![](images/2020-08-11-16-33-50.png)
![](images/2020-08-11-16-33-59.png)


## 15. MapReduce Summary Quiz
- [X] single Pass
- [ ] Multiple pass
---------
- [ ] Skewed distribution of keys
- [X] Uniformly-distribution of keys
---------
- [X] No synchronization needed
- [ ] A lot of synchronization needed