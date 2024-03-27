# Introduction 
All of the products people use on a daily basis are manufactured. There is great potential for data mining to reduce waste in manufacturing by optimizing the time it takes to produce products and reducing the number of manufacturing defects. These improvements benefit companies, consumers and the environment. Optimizing the cycle time of production processes allows companies to create the same products in less time with less labor and equipment costs while reducing manufacturing defects lowers the number of units that need to be discarded which has a clear cost impact. Both of these improve the profits of companies. Consumers benefit because products can cost less and be of higher quality. Lowering the energy and raw material requirements by reducing waste in manufacturing improves the environmental impact of a product. 
Manufacturing innovations have driven human history and data science, including data mining, has played a critical role in many innovations. We propose a new tool to identify cycle times and predict failures based on real-time manufacturing data and with a level of granularity not feasible without data mining techniques. Theoretical models have been used extensively to optimize manufacturing processes. This project will close the gap between theory and reality by enabling actionable insights based on real-time data that are not limited by the abstraction of theoretical models. 

## Objective 
The objective of this project is to create a classification model using synthesized manufacturing data. This can be usedretrospectively by operation optimization specialists and
manufacturing experts to optimize their assembly lines and learnwhere costs can be reduced.To meet this objective, there will be three main phases;synthesizing the data to simulate a real-world manufacturing line,creating an anomaly detection model, and creating a binaryclassification model determining which units meet the qualitystandard and which do not.With this final model, stakeholders will use feature importancevalues be able to understand the underlying factors that influencemanufacturing failures the most.

## Proposed Solution
The solution architecture for this problem contains three key stages: data synthesis, building an anomaly detection model, and
building a classification model. Each of the models involves preprocessing the data for model consumption and visualizing the
results of the model after tuning hyper-parameters.The output of this model can be utilized by manufacturing
management experts to analyze flagged units to create a more
efficient and lean manufacturing system.

## Methodology

### Data Synthesis
The first stage of the solution is the data synthesis; the objective isto mimic data collection similar to that would be found in amanufacturing facility. The final data synthesis program has four main parts:
- Basic Assembly line setup
- Cycle Time
- Parametric Data
- Failure Injection

### Anomaly Model Data Preprocessing
Once the synthesized data is obtained, substantial datapreprocessing steps need to be taken to prepare the dataset to beincorporated in the anomaly detection model. We implementedthese data preprocessing steps to ensure data quality, transformthe dataset to be more compatible with our anomaly detection
model, and introduce new features that enhance our model’s
prediction capabilities.
The first data preprocessing step that was taken was to read the
synthesized data which was exported as a CSV and turn it into a
Pandas DataFrame. This allowed us to transform the dataset and
perform all necessary data preprocessing steps until the newly
transformed dataset was exported as a CSV to our anomaly
detection model. Once our data was converted into a DataFrame,
we removed the first column which was an unintended output of
the data synthesizer and had no relevant information for the
model. Furthermore, unrecorded measurements for records in the
dataset were denoted as “None” of type string, which was
converted to NoneType. After the missing values were converted
to NoneType, we imputed those missing values with the most
recent measurements that occurred for that unit in the
manufacturing line. With the measurement column now updated,
we included 3 lag columns for the measurement feature to
enhance the model’s interpretation of anomalous measurements
by providing historical context for measurement values.
The next step in our process was to calculate the cycle time,
which represented the time spent by each unit at each process. To
calculate this time, we first had to convert our Unit_ID and
Process_ID features into a numeric value, Group the dataset by
Unit_ID, sort the dataset by Process_Id and Timestamp, and lastly
find the difference between each timestamp for each unit. By
calculating cycle time this way, we were left with null values for
the first instance of each unit in the manufacturing line. To correct
this issue, we filtered the dataset by the first instance of each unit
and then calculated their respective cycle times by finding the
difference between the last timestamp of the previous unit in the
manufacturing line and the first instance of the current unit in the
manufacturing line. This still led to cycle time issues for the first
unit and the last unit that entered the manufacturing line, but we
excluded the first and last units present in the dataset to remove
the ramp-up and ramp-down times because they do not accurately
reflect the manufacturing process in its standard state.
Afterward, we added a new column for the average measurement
at each process and the average cycle time at each process. With
these two new columns, we were able to include two additional
columns that represented the absolute difference between the
measurement and cycle time of that record to the average
measurement and cycle time of the process. This additional
information provided a useful feature for examining whether or
not the measurements and times for that record were anomalous in
comparison to the average for that process.
Once this step was completed, we one-hot encoded the categorical
features for operator and process ID and dropped the original
column to avoid multicollinearity. This allowed the categorical
columns to become numerical features so that they could be
included in the model. Then, we moved on to scaling our data,
employing min-max scaling to transform our numerical features
into a range of [0, 1]. Scaling the data was an important step
because our model used support vector machine, a distance-based
algorithm, and we wanted to ensure that all features would be
interpreted with equal importance and that the model would not be
as sensitive to outliers.
Lastly, we joined the categorical features with the newly scaled
numeric features and exported the DataFrame as a CSV so it
could be implemented in our anomaly detection model.

### Anomaly Model Building
The model chosen for anomaly
detection was one-class SVM, mainly due to its strength in
dealing with class imbalance.
2.2.2 Model Execution. To develop the anomaly detection model,
we used GridSearchCV to tune the hyperparameters. In order to
view feature importance, the kernel used for the SVM was linear.
Since one-class support vector machine is an unsupervised model,
there was only the training dataset. This was used to extract the
confusion matrix and feature importance of the model.
The goal of this model was to extract a feature that indicates the
probability or degree to which each record could be anomalous for
use in the classification model. To do this, we used the scores that
SVM provides and concatenated that with the dataset for
preprocessing of the next stage. However, there were still some
model accuracy and performance results analyzed to ensure that
the model was providing value.

### OK/NG Classification Model Data
After the anomaly detection model is completed and anomaly
detection scores are created for each unit at each process, the
output of the model requires further preprocessing until it can be
implemented into our classification model where we can identify
which units are “Ok” or “No Go”.
The first preprocessing step we took was to transform the data by
making the ID for each unit the primary key for the dataset.
Having only one instance of each unit enabled our classification
model to classify each unit one time instead of predicting each
unit at each process. To begin, we removed the additional
columns from the anomaly detection model output that had no
value for the classification model. After that was completed, we
once again one-hot encoded the Operator_ID column as it had
been reverted back to categorical after we updated the dataset to
contain only one unit in each row. Once this transformation was
completed, we renamed each of the columns in the updated
dataset to accurately reflect the value they contained. Many null
values arose when merging each instance of a unit into a single
record and in response we imputed those null values with 0 to
represent an instance where a measurement or an anomaly had not
taken place.
Our next step was to use the multiple anomaly detection scores for
each unit and create an average anomaly detection score, which
was used as a new feature for the classification model while the
other scores were dropped from the dataset. After the average
anomaly detection scores were calculated for each unit, the scores
were normalized to be on the same scale as all of the other
features ([0, 1]) to improve the model's effectiveness.
Additionally, we calculated the mean and standard deviation of
the last measurement recorded for each unit. From this, we created
an upper and lower boundary of this average measurement that
determined the bounds of acceptable parametric measurements. It
identified any final measurements for a unit that exceeded the
boundary thresholds. These values were included in a new feature
called “pass” which was 1 if the measurement was within the
boundaries and 0 otherwise. This enabled our model to identify
what anomalous measurements could fail a unit and in tandem
with the score derived from the anomaly model we introduced
important new features that made the classification of failed units
exponentially more accurate. Finally, after the preprocessing steps
were completed, we exported the DataFrame as a CSV so it could
be implemented into the classification model.

### OK/NG Classification Model Building
Based on this comparison, the model chosen for the classification
model was XGBoost, mainly due to the high dimensionality of the
data and ambiguous relationship between the features and target. To develop the classification model, we
used GridSearchCV to tune the hyperparameters. Since this is also
a binary classification exercise, we generated a confusion matrix
and respective metrics from that.


