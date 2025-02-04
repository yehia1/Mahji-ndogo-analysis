# Maji Ndogo

## Introduction

This project aims to investigate the water source of the maji ndogo area

project in alx data science course

## Data Schema

![image](./Screenshot%202024-04-20%20204855.png)

The data consists of 8 tables connected to each other with 1 to many or one to one one

### Tables

#### 1. water_source
- **source_id:** unique id of each source
- **Type of water source:** The type of water which includes well, tap, tap in home
- **Number of people served:** The number of people that can access this water source
- **Average\_queue\_time:** Calculated field in power BI to know average time for each set of people to reach the water source
- **Basic\_water\_access:** Calculated column to see if set of people can access basic water service

#### 2. Well Pollution

- **source_id:** unique id of each source
- **results:** the result of investing about pollution in each source

#### 3. Visits 
- **source_id:** unique id of each source
- **record_id:** the record of each visit happend to a water source 
- **location_id:** location of each visit
- **visit_count:** number of visits to a place with same source
- **time_in_queue:** time it takes in this source in this location to access the source.

#### 4. Vendors 
- **vendor_id:** This unqiue value of each vendor
- **company_name:** The name that represents each vendor
- **owner_name:** The owner of the vendor
- **company_type:** The type of work the company do.

#### 5. infrastructure_cost
- **improvement:** The type of improvment for each source 
- **unit_cost_USD:** The cost of each improvment
- **Rural_cost_USD:** The increase cost for the rural areas.

#### 6. Location
- **location_id**: unique id of location
- **province_name:** The province of the location
- **town_name:** The town of the location 
- **location_type:** rural or urban

#### 7.project progress
- **project_id:** Unique id of the project 
- **source_id:** Unique id of the water source
- **town:** The town of each project
- **source_type:** Source type of water
- **improvement:** Improvment required for each water source
- **source_status:** tells if complete or not
- **Aggregated_improvements** : calculated column adds unknown type of improvment to a known one
- **Budgeted_improvement_cost:** Calculated field for differntiate rural and urban costs.
- **date_of_completion:** The date to complete the project 
- **assigned_vendor:** vendor who will make the project
- **date_started:** Date of the start of the project 
- **cost:** Cost of the projcet.

## Power BI Measures
### project progress
All in dax 
```dax
Completed = CALCULATE( COUNTROWS(project_progress),
    project_progress[source_status] == "Complete")
```
Count the number of completed projects

```
cumulative_budget = 
CALCULATE(
    SUM('project_progress'[Budgeted_improvement_cost]),
    FILTER(
        ALL('project_progress'[date_of_completion]),
        'project_progress'[date_of_completion] <= MAX('project_progress'[date_of_completion]) &&
        NOT(ISBLANK('project_progress'[date_of_completion])) // This line removes blank\null values.
        )
)
```
calcualte the cumulative budget of the completed projects

```
cumulative_cost = 
CALCULATE(
    SUM('project_progress'[cost]),
    FILTER(
        ALL('project_progress'[date_of_completion]),
        'project_progress'[date_of_completion] <= MAX('project_progress'[date_of_completion]) &&
        NOT(ISBLANK('project_progress'[date_of_completion])) // This line removes blank\null values.
        )
)
```
calcualte the cumulative cost of the completed projects

```
pct_project_complete = [Completed] / COUNTROWS(All(project_progress))
```
calcuate the percent of completed projects

```
Sources_to_go = [total_improvements] - [Completed]
```
number of sources left to finish the projects

```
total_improvements = 
    CALCULATE(
        COUNTROWS('project_progress'),
        ALLEXCEPT('project_progress','project_progress'[town]))
```
used for filtering by town

### Water source

```
Basic_Acess_percentage = 
    ([population_now_basic_access] + [population_with_basic_access]) / [total_population]
```
calcuate the percentage of access by time.


```
population_now_basic_access = 
    CALCULATE(
        SUM(water_source[number_of_people_served]),
        project_progress[source_status] = "Complete"
    )
```
The sum of people served by projects completed 

```
population_with_basic_access = 
    CALCULATE( // Calculate with some conditions
        SUM('water_source'[number_of_people_served]),// Population when....
        FILTER(
            ALL(water_source),
            OR(// Nested or to have well, OR tap_in_home OR shared tap
                OR(
                AND( // When it is a well, it must be clean too
                'water_source'[type_of_water_source] = "well",
                RELATED(well_pollution[results]) = "Clean"
                ),
                'water_source'[type_of_water_source] = "tap_in_home"
                ),
                AND( // When it is a shared tap, it must have a short queue time
                'water_source'[type_of_water_source] = "shared_tap",
                'water_source'[Average_queue_time] < 30
                )
            )
        )
    )
```
Sum of people with basic access in the moment.

```
total_improvment = 
    SUMX(
        FILTER(
            water_source,
            RELATED(project_progress[source_id]) = water_source[source_id]
        ),
        water_source[number_of_people_served]
    )
```
Number of improvement left to make

```
total_population = 
    CALCULATE(
        Sum('water_source'[number_of_people_served]),
        ALLEXCEPT(
        'project_progress',
        'project_progress'[town]
        )
        )
```
Calcuate the total people and made it filtered by town

```
Unaccesed_percentage = CONCATENATE("+", ROUND((1 -  [Basic_Acess_percentage])* 100,0) & "%" )
```
Used in analysis to show + or - in the label.

## DashBoard Snipps

![image](./Screenshot%202024-04-20%20212715.png)

![image](./Screenshot%202024-04-20%20212842.png)

![image](./Screenshot%202024-04-20%20212928.png)

## Conclusion
by making this analysis we were able to detect the gaps and places with low water access and made the project for all the towns and provinces and serve people there to make all the improvments required.

