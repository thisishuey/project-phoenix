# BigQuery Table and Schemas

## BigQuery Tables

The system writes data to the following BigQuery tables:

1. [Desk Bookings Table](#Desk_Bookings_Table)
2. [Desk Moves Table](#Desk_Moves_Table)
3. [Desk Shifts Table](#Desk_Shifts_Table)
4. [Desks Table v2](#Desks_Table_v2)
5. [Employees Table](#Employees_Table)
6. [Greetly Visits Table](#Greetly_Visits_Table)
7. [Neighborhoods Table](#Neighborhoods_Table)
8. [Presence Table](#Presence_Table)
9. [Sites Table](#Sites_Table)

## Desk_Bookings_Table

| Field Name          | Data Type  | Mode      | Description                                    |
|:--------------------|:-----------|:----------|:-----------------------------------------------|
| Client_Id           | STRING     | REQUIRED  | Organization identifier                        |
| Desk_Booking_Id     | STRING     | REQUIRED  | Unique identifier for the booking              |
| Desk_Id             | STRING     | REQUIRED  | Reference to the desk being booked             |
| Series_Id           | STRING     | NULLABLE  | Identifier for recurring bookings              |
| Employee_Id         | STRING     | NULLABLE  | Reference to the employee making the booking   |
| Check_In_Time       | TIMESTAMP  | REQUIRED  | When the booking starts                        |
| Check_Out_Scheduled | TIMESTAMP  | REQUIRED  | When the booking is scheduled to end           |
| Check_Out_Time      | TIMESTAMP  | NULLABLE  | When the booking actually ended                |
| Confirmed_Time      | TIMESTAMP  | NULLABLE  | When the booking was confirmed                 |
| Completed_Time      | TIMESTAMP  | NULLABLE  | When the booking was marked as completed       |
| Canceled_Time       | TIMESTAMP  | NULLABLE  | When the booking was canceled (if applicable)  |
| Visit_Id            | STRING     | NULLABLE  | Optional reference to a related visit record   |
| Effective_Time      | TIMESTAMP  | REQUIRED  | When this record became effective              |
| Day                 | DATE       | NULLABLE  | Calendar date of the booking                   |

## Desk_Moves_Table

| Field Name      | Data Type  | Mode      | Description                                    |
|:----------------|:-----------|:----------|:-----------------------------------------------|
| Client_Id       | STRING     | REQUIRED  | Organization identifier                        |
| Move_Id         | STRING     | REQUIRED  | Unique identifier for this move                |
| From_Desk_Id    | STRING     | NULLABLE  | Reference to the source desk                   |
| To_Desk_Id      | STRING     | NULLABLE  | Reference to the destination desk              |
| Employee_Id     | STRING     | REQUIRED  | Reference to the employee being moved          |
| Scheduled_Time  | TIMESTAMP  | REQUIRED  | When the move is planned to occur              |
| Completed_Time  | TIMESTAMP  | NULLABLE  | When the move was actually completed           |
| Canceled_Time   | TIMESTAMP  | NULLABLE  | When the move was canceled (if applicable)     |
| Effective_Time  | TIMESTAMP  | REQUIRED  | When this record became effective              |

## Desk_Shifts_Table

| Field Name                | Data Type  | Mode      | Description                                    |
|:--------------------------|:-----------|:----------|:-----------------------------------------------|
| Client_Id                 | STRING     | REQUIRED  | Organization identifier                        |
| Desk_Availability_Id      | STRING     | REQUIRED  | Identifier for the availability definition     |
| Shift_Id                  | STRING     | REQUIRED  | Unique identifier for this shift               |
| Desk_Id                   | STRING     | REQUIRED  | Reference to the desk                          |
| Desk_Availability_Name    | STRING     | REQUIRED  | Human-readable name for this availability pattern |
| Desk_Availability_Color   | STRING     | REQUIRED  | Hexadecimal color code for UI display          |
| Desk_Availability_Type    | STRING     | NULLABLE  | Pattern type (e.g., "ALWAYS", "DAYS_OF_WEEK", "WEEKS_OF_YEAR") |
| Desk_Availability_Values  | INT64      | REPEATED  | Array of integers representing the pattern values |
| Deleted                   | BOOL       | REQUIRED  | Whether this shift definition has been removed |
| Effective_Time            | TIMESTAMP  | REQUIRED  | When this record became effective              |

## Desks_Table_v2

| Field Name            | Data Type  | Mode      | Description                                    |
|:----------------------|:-----------|:----------|:-----------------------------------------------|
| Effective_Time        | TIMESTAMP  | REQUIRED  | When this record became effective              |
| Day                   | DATE       | REQUIRED  | The date associated with this desk state       |
| Client_Id             | STRING     | REQUIRED  | Organization identifier                        |
| Site_Id               | STRING     | REQUIRED  | Location identifier                            |
| Site_Name             | STRING     | NULLABLE  | Human-readable site name                       |
| Floor_Id              | STRING     | REQUIRED  | Identifier for the floor where desk is located |
| Floor_Name            | STRING     | NULLABLE  | Human-readable floor name                      |
| Desk_Id               | STRING     | REQUIRED  | Unique identifier for the desk                 |
| Desk_Name             | STRING     | NULLABLE  | Human-readable desk name                       |
| Desk_Deleted          | BOOL       | REQUIRED  | Whether this desk has been removed             |
| Desk_Active           | BOOL       | REQUIRED  | Whether this desk is currently in service      |
| Desk_Assignment_Type  | INT64      | NULLABLE  | Numerical code for desk assignment type        |
| Desk_Shifts           | STRUCT     | REPEATED  | Nested structure for desk shift information    |
| Site_Time_Zone        | STRING     | NULLABLE  | Time zone identifier for the site              |
| Floor_Online          | BOOL       | NULLABLE  | Whether the floor is currently online          |
| Neighborhood_Id       | STRING     | NULLABLE  | Optional grouping of related desks             |

### Desks_Table_v2 > Desk_Shifts

The **Desk_Shifts** field is a nested structure under **Desk_Table_v2** with:

| Field Name  | Data Type  | Mode      | Description                                    |
|:------------|:-----------|:----------|:-----------------------------------------------|
| Type        | STRING     | NULLABLE  | Indicates the shift type                       |
| Values      | INT64      | REPEATED  | Array of values associated with the shift      |

## Employees_Table

| Field Name        | Data Type  | Mode      | Description                                    |
|:------------------|:-----------|:----------|:-----------------------------------------------|
| Effective_Time    | TIMESTAMP  | REQUIRED  | When this record became effective              |
| Client_Id         | STRING     | REQUIRED  | Organization identifier                        |
| Id                | STRING     | REQUIRED  | Unique employee identifier                     |
| First_Name        | STRING     | NULLABLE  | Employee's first name                          |
| Last_Name         | STRING     | NULLABLE  | Employee's last name                           |
| External_Id       | STRING     | NULLABLE  | External system identifier                     |
| Cost_Center       | STRING     | NULLABLE  | Financial grouping                             |
| Department        | STRING     | NULLABLE  | Organizational department                      |
| Team              | STRING     | NULLABLE  | Team within department                         |
| Title             | STRING     | NULLABLE  | Job title                                      |
| Assigned_Site_Id  | STRING     | NULLABLE  | Reference to employee's assigned site          |
| Active            | BOOL       | NULLABLE  | Whether the employee is active                 |
| Start_Date        | DATE       | NULLABLE  | Employment start date                          |
| End_Date          | DATE       | NULLABLE  | Employment end date (if applicable)            |

## Greetly_Visits_Table

| Field Name          | Data Type  | Mode      | Description                                    |
|:--------------------|:-----------|:----------|:-----------------------------------------------|
| Event_Start_Time    | TIMESTAMP  | REQUIRED  | When the visit started                         |
| Day                 | DATE       | REQUIRED  | Calendar date of the visit                     |
| Client_Id           | STRING     | REQUIRED  | Organization identifier                        |
| Check_In_Record_Id  | STRING     | REQUIRED  | Unique identifier for the check-in record      |
| Site_Id             | STRING     | REQUIRED  | Location where the visit occurred              |
| Employee_Id         | STRING     | NULLABLE  | Employee being visited                         |
| Visit_Type          | STRING     | REQUIRED  | Purpose or category of visit                   |
| Visitor_First_Name  | STRING     | NULLABLE  | Visitor's first name                           |
| Visitor_Last_Name   | STRING     | NULLABLE  | Visitor's last name                            |
| Visitor_Email       | STRING     | NULLABLE  | Visitor's email address                        |
| Visitor_Id          | STRING     | NULLABLE  | Unique identifier for the visitor              |

## Neighborhoods_Table

| Field Name      | Data Type  | Mode      | Description                                    |
|:----------------|:-----------|:----------|:-----------------------------------------------|
| Client_Id       | STRING     | REQUIRED  | Organization identifier                        |
| Id              | STRING     | REQUIRED  | Unique neighborhood identifier                 |
| Name            | STRING     | REQUIRED  | Human-readable name of the neighborhood        |
| Description     | STRING     | NULLABLE  | Optional description                           |
| Deleted         | BOOL       | REQUIRED  | Whether this neighborhood has been removed     |
| Floor_Id        | STRING     | REQUIRED  | Reference to the floor containing this neighborhood |
| Effective_Time  | TIMESTAMP  | REQUIRED  | When this record became effective              |

## Presence_Table

| Field Name        | Data Type  | Mode      | Description                                    |
|:------------------|:-----------|:----------|:-----------------------------------------------|
| Effective_Time    | TIMESTAMP  | REQUIRED  | When this record became effective              |
| Day               | DATE       | REQUIRED  | Calendar date of the presence event            |
| Client_Id         | STRING     | REQUIRED  | Organization identifier                        |
| Site_Id           | STRING     | REQUIRED  | Location identifier                            |
| Site_Name         | STRING     | NULLABLE  | Human-readable site name                       |
| Site_Time_Zone    | STRING     | NULLABLE  | Time zone identifier for the site              |
| Presence_Event_Id | STRING     | NULLABLE  | Unique identifier for the presence event       |
| Device_Id         | STRING     | NULLABLE  | Identifier for the device that recorded the presence |
| Ingestion_Type    | STRING     | REQUIRED  | How the presence data was collected            |
| Employee          | STRUCT     | REQUIRED  | Nested structure with employee information     |
| Deleted           | BOOL       | NULLABLE  | Whether this record has been removed           |

### Presence_Table > Employee

The **Employee** field is a nested structure under **Presence_Table** with:

| Field Name    | Data Type  | Mode      | Description                                    |
|:--------------|:-----------|:----------|:-----------------------------------------------|
| Id            | STRING     | NULLABLE  | Employee identifier                            |
| External_Id   | STRING     | NULLABLE  | External system identifier                     |
| Department    | STRING     | NULLABLE  | Organizational department                      |
| Cost_Center   | STRING     | NULLABLE  | Financial grouping                             |
| Team          | STRING     | NULLABLE  | Team within department                         |
| Title         | STRING     | NULLABLE  | Job title                                      |
| First_Name    | STRING     | NULLABLE  | Employee's first name                          |
| Last_Name     | STRING     | NULLABLE  | Employee's last name                           |

## Sites_Table

| Field Name     | Data Type  | Mode      | Description                                    |
|:---------------|:-----------|:----------|:-----------------------------------------------|
| Effective_Time | TIMESTAMP  | REQUIRED  | When this record became effective              |
| Client_Id      | STRING     | REQUIRED  | Organization identifier                        |
| Id             | STRING     | REQUIRED  | Unique site identifier                         |
| Name           | STRING     | NULLABLE  | Human-readable site name                       |
| Deleted        | BOOL       | NULLABLE  | Whether this site has been removed             |
| Time_Zone      | STRING     | NULLABLE  | Time zone identifier for the site              |