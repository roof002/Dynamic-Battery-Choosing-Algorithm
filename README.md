# Dynamic-Battery-Choosing-Algorithm

## Context of Code
To build an algorithm to select the best combinations of 5 batteries (Batt1, Batt2, Batt3, Batt4, Batt5) that can be used to reduce the total load of 5 sites (Site1, Site2, Site3, Site4, Site5) during an activation period (from StartTime to EndTime of ActivationNotice).
- Each battery corresponds to a site (Batt1 -> Site1, Batt2 -> Site2, etc.)
> Note: Sometimes batteries are labeled BESS[i], take BESS == Batt

## Tables used from MySQL Database
### ActivationNotice 
This table records ReceivedTime, TotalLoad, StartTime, EndTime, Type
- **ReceivedTime** (_YYYY-MM-DD HH:MM:SS_, timestamp): Time when user submits a request of activation on the website 
- **TotalLoad** (_kW/h_, float): Rate requested to reduce (for an hour)
- **StartTime** (_YYYY-MM-DD HH:MM:SS_, timestamp): Start time of the activation requested
- **EndTime** (_YYYY-MM-DD HH:MM:SS_, timestamp): End time of the activation requested
- **Type** (_DR/IL_, str): The type of request the user wants with the activation

### BatteriesTable
This table records ReadTime, BESS1_Status, BESS1_Capacity, BESS2_Status, BESS2_Capacity, BESS3_Status, BESS3_Capacity, BESS4_Status, BESS4_Capacity, BESS5_Status, BESS5_Capacity
- **ReadTime** (_YYYY-MM-DD HH:MM:SS_, timestamp): Updates every minute of changes with each battery's capacities
- **BESS[i]_Status** (int): For i = 1,2,3,4,5, this updates the status of the batteries at every minute (-1: Discharging, 0: No change, 1: Charging), based on what is sent in SitesCommand (see below)
- **BESS[i]_Capacity** (_kWh_, float): For i = 1,2,3,4,5, each BESS has a different capacity volume
  - BESS1 = 1250 kWh
  - BESS2 = 1000 kWh
  - BESS3 = 625 kWh
  - BESS4 = 625 kWh
  - BESS5 = 500 kWh

### SitesCommand
This table records ReceiveTime, Site1_CnD, Site1_Amt, Site2_CnD, Site2_Amt, Site3_CnD, Site3_Amt, Site4_CnD, Site4_Amt, Site5_CnD, Site5_Amt
- **ReceiveTime** (_YYYY-MM-DD HH:MM:SS_, timestamp): Updates whenever there's a command sent
- **Site[i]_CnD** (int): For i = 1,2,3,4,5, this updates the status of the batteries at ReceiveTime (-1: Discharging, 0: No change, 1: Charging)
- **Site[i]_Amt** (_kW/h_, float): For i = 1,2,3,4,5, a rate in terms of kW/h is sent for batteries that algorithm picks. Discharging limits are as follows:
  1. Site1: 450kW/h - 500kW/h
  2. Site2: 350kW/h - 400kW/h
  3. Site3: 200kW/h - 250kW/h
  4. Site4: 200kW/h - 250kW/h
  5. Site5: 150kW/h - 200kW/h    

### SitesPowerNow
This table records ReadTime, Date, Period, Site1, Site2, Site3, Site4, Site5
- **ReadTime** (_YYYY-MM-DD HH:MM:SS_, timestamp): Updates every 1min to 1min 1sec for new data on each site
- **Date** (_YYYY-MM-DD HH:MM:00_, timestamp): Rounds down each record to the minute
- **Period** (int): Splits every 30mins into 1 Period, for 48 Periods
- **Site[i]** (_kW/h_, float): For i = 1,2,3,4,5, this updates the rate of battery used to run each Site
