
<img src ="https://user-images.githubusercontent.com/89044350/152267296-ad42e344-9cba-4295-a1e4-db557a73dcb4.jpg" width = '1000'>

# Visualizing Moon Phases and Crime Types in Gainesville, Florida

# Overview
Everyone has heard the tale of weird things happening during a full moon. Police, fire rescue, service operators and hospitals are speculated to see a spike in calls during a full moon phase. This project attempts to analyze the calls for service and associated crimes in Gainesville, Florida that went to the police department between 2018 and 2021 during the four moon phases (New Moon, First Quarter, Third Quarter, and Full Moon).  We selected this topic to explore if there is any connection to the moon phases and calls to service to determine if the myth is simply a myth or a probable explanation for the weird things that occur on a full moon.

H₀= There is no statistical difference between the moon phase and crime type.

Ha=There is a statistical difference between moon phase and crime type.

In this model the main question we aim to answer:

1. Can we predict the type of crimes based on the moon phases? 
 Larger Analysis Questions:
2. What type of crimes should we expect leading up to the full moon?
3. Are the moon phases relevant? If so, which is the most?


If you’re looking to reproduce this project, please take a look at “working_folder”.

[Presentation](https://docs.google.com/presentation/d/11eIVAccX1Z8nAHaC2epMZFV1IC3TIXff1RNx4FilYmQ/edit?usp=sharing)

## Data Source
This project uses two sets, one from City of Gainesville [website](https://data.cityofgainesville.org/Public-Safety/Crime-Responses/gvua-xt9q) and Moon Phase dates from [here](https://weather.visualcrossing.com/VisualCrossingWebServices/rest/services/timeline/Gainesville,FL/2018-01-01/2021-12-31?unitGroup=us&key=JVFDPCT4LWWPVKADN783XGRVA&include=days&elements=datetime,moonphase).

The City of Gainesville data is a csv file with unique crime IDs, incident type, Report Date, Offense Date, Report Hour of Day, Report Day of Week, Offense Hour of Day, Offense Day of Week, City, State, Address, Latitude, Longitude, and Location. The data from this source represents calls for service where a report was written. The moon phase data source is a JSON object for a single location in Gainesville, FL with the moon phase and date for every date between 2018-2021. The keys are "datetime" and "moon phase". Datetime is in year/month/day format and each moon phase value corresponds to a positive value between the numbers 0-1. A New Moon = 0, First Quarter =0.25, Third Quarter=.75, and Full Moon=1.

The selected moon phase date range for this project was January 1st, 2018, to December 31st, 2021.

# Preprocessing

## Data Cleaning

We got rid of incidents that didn’t necessarily imply a crime occurred and refined our crime types to the unique list of instances below.
Unique incident types that were removed: 
1. assist other agencies,
2. assist citizen
3. warrant arrest
4. lost/stolen vehicle tag/ decal
5. DCF investigation
6. Drug possession of controlled substance
7. information
8. found contraband
9. tow report 
10. recovered stolen vehicle 
11. stalking (simple)
12. found-returned
13. seize tag


Our data source was cleaned using Jupyter Notebook and Pandas. First we renamed our columns as such:

![image](https://user-images.githubusercontent.com/87162266/153163567-160b4ef8-0adf-4b22-91cc-62f8360e3bd6.png)


After renaming our columns, we filtered the crimes on 01-01-2018 - 12-31-2021 on the "offenseDate" column. Next, we dropped all columns but "ID","CFS", "offenseDate", "offenseHour", "offenseDOW", "latitude", "longitude" and "CFS_Type". 

The moon phase data source was read in using an API call as a data frame. The date-time format was changed with pandas "to_datetime".  The moon phase and Gainesville Crime and Classifications data frames were all merged on the Date column. Persons (violent), Property(non-violent), Government and other are the four crime classifications ("CFS") a crime type ("CFS_Type") could fall under. 

With the remaining unique incident types, we decided to group them into the following categories: 

![image](https://user-images.githubusercontent.com/87162266/153163988-d011a071-be11-49d4-8b7e-52bec7888577.png)

- The 10 crime types were then separated and classified as either Person or Property crimes with Python using str.contains. And then, the LabelEncoder from sklearn was used to encode the "class", "moonphase", "offensedow", "cfs", "date", and "cfs_type" columns.

## Data Storage

### ERDiagram

![ERD2](https://user-images.githubusercontent.com/87162266/151726526-97a0ace3-53e2-45fe-9e45-6a7fb43842f7.png)

Date is the foreign key connection between CallsForService and MoonPhase. "CFS" is the foreign key connection to classification from CallsForSerivice.

Once our data was cleaned we used pgAdmin to created our SQL database. After naming the databases we created three tables with this [SQL file](https://github.com/DWashington3/FinalProject/blob/d681d81ef7e4160ad2964b83ed3aa4d78480b917/docs/CreateTables.sql)

Following the table creation we imported our data into the tables. This allowed us to make a connection in jupyter notebook. 

# Machine Learning
SciKitLearn is the library that will be used. Before feeding our information into the model we used the LabelEncoder to convert our string data types to floats. We defined our X and Y as follows:
 - X = ['cfs', 'date', 'offensedow', 'cfslatitude,'cfslongitude'.'cfs_type', 'moonphase']
 - y = ['cfs_class']
The model is then fit using RandomForestClassifier, Scaled using StandardScaler() then fit using a confusion_matrix, accuracy_score, classification_report. 
  
## Results

Our model returned results that many myth believers will be sad to hear. 
  The Predicted Acutal and Predicted Person or Property Crime
  
  ![image](https://user-images.githubusercontent.com/87162266/153167592-568dab0c-9434-4b07-9059-8d3371326a2b.png)

  
  The Confusion Matrix
  
  ![image](https://user-images.githubusercontent.com/87162266/153168497-5fb268a6-c8f0-4500-babc-bf87242f5b09.png)
      
  The Feature Importance
  
   ![image](https://user-images.githubusercontent.com/87162266/153167430-04580cdd-ac02-4578-831a-85da90243c88.png)


# Visualuations 
We used Tableau Public for visual displays and a fully interactive [Dashboard](https://public.tableau.com/app/profile/jake.wolfe/viz/Gainesville_Crime_Project/InteractiveDashboard?publish=yes)


### Screenshot of Interactive Dashboard
<img src = "https://user-images.githubusercontent.com/89044350/153247362-f2e3df88-5ea3-44cf-bfb3-6f5f91e7c0eb.JPG">

The dashboard allows users to make preselected conditions (crime classification and crime type) automatically updating the map to display the location in Gainesville, and total counts of the selected crime(s).  If a user should then select a bubble within the map, the lower bar chart will display the total amount of the selected crime per the 4 major moon phases.

### Moon Phases vs CFS Type Totals
<img src = "https://user-images.githubusercontent.com/89044350/153249222-a687036c-e14b-47d2-ba07-89006436e0c5.JPG">

The above image breaks down each Call For Service (Crime) type total, across the four major moon phases.  As you can see, theft is the most committed crime across all moon phases.  This chart also suggests that moon phase has no affect on which crime is committed, or to the overall sum of crimes.

### Crimes by Hour and Moon Phase
<img src = "https://user-images.githubusercontent.com/89044350/153251854-2975bf48-a0a8-4dca-b381-bea8e224f1ae.JPG">

  
  
# Conclusion

There is no statistical significance between the moon phase and crime type. Our model concluded moon phases weren't an important feature when determining whether a crime was a person (violent) crime or property (non-violent) crime. The model didn't necessarily convey meaningful information. After looking at the accuracy score of 99%, it is clear that our model was over-fitted for our data. Improvements to the  model will allow for more prediction outputs. With more predictions we could evaluate the fidelity and feasibility of our model.  Our improved model  would allow better allocated police and emergency response resources, saving the government a potential hundreds of thousands of dollars a year. An improved model will allow us to trend what crimes to expect leading up to a certain moon phase- in this case -a full moon. At this point we cannot conclude if moon phases have any affect on crime and furthermore which moon phase is the most relevant. 
