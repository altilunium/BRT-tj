# BRT-tj
BRT Transjakarta

![image](https://github.com/altilunium/BRT-tj/assets/70379302/c6fb1029-12cc-423c-99b3-1f947a74df09)


[https://altilunium.github.io/brt-tj](https://altilunium.github.io/BRT-tj/)


### Data source

https://gtfs.transjakarta.co.id/files/file_gtfs.zip (Downloaded : January 26th, 2024, 21.18 GMT + 7)

### How it works
First, extract the file_gtfs.zip file. Then, create a Python script to combine all the information needed to display Transjakarta's BRT routes and stops. This information is scattered across routes.txt, trips.txt, shapes.txt, stop_times.txt, and stops.txt. Package all of this information into a single JSON file. 

```python
import csv
import json

brts = dict()

# Process routes.txt
# Filter Transjakarta BRT by using "row[4] == 'BRT'"
def iterate_csv(file_path):
    with open(file_path, 'r') as file:
        reader = csv.reader(file)
        header = next(reader)  
        
        for row in reader:
            rid = row[0]
            column2 = row[3]
            # ...
            if (row[4] == 'BRT'):
               brts[row[0]] = dict()
               brts[row[0]]['name'] = row[3]
               brts[row[0]]['trips'] = []
               brts[row[0]]['tripsname'] = []
               brts[row[0]]['tripsid'] = []
file_path = 'routes.txt'
iterate_csv(file_path)

# Process trips.txt
with open('trips.txt', 'r') as file:
        reader = csv.reader(file)
        header = next(reader) 
        for row in reader:
            rid = row[0]
            column2 = row[3]
            if (row[0] in brts):
                brts[row[0]]['trips'].append(row[6])
                brts[row[0]]['tripsname'].append(row[1]+":"+row[3])
                brts[row[0]]['tripsid'].append(row[2])


# Reference dictionary for routes and trips
routes = dict()
for i in brts:
    for j in brts[i]['trips']:
        print(j)
        routes[j] = []

trips = dict()
for i in brts:
    for j in brts[i]['tripsid']:
        trips[j] = []


# Process shapes.txt
with open('shapes.txt', 'r') as file:
        reader = csv.reader(file)
        header = next(reader) 

        for row in reader:
            rid = row[0]
            column2 = row[3]
            if (row[0] in routes):
                print(row[0])
                routes[row[0]].append([row[1],row[2]])

# Process stop_times.txt
stops = dict()
with open('stop_times.txt', 'r') as file:
        reader = csv.reader(file)
        header = next(reader) 
        
        for row in reader:
            rid = row[0]
            column2 = row[3]
            if (row[0] in trips):
                trips[row[0]].append(row[3])
                stops[row[3]] = dict()


# Process stops.txt
with open('stops.txt', 'r') as file:
        reader = csv.reader(file)
        header = next(reader) 
        for row in reader:
            rid = row[0]
            column2 = row[3]
            if (row[0] in stops):
                stops[row[0]]["name"] = row[2]
                stops[row[0]]["coords"] = [row[4],row[5]]
 

# Package everything to a single data.json
packaged = []
for i in brts:
    a_route = dict()
    print(brts[i]["name"])
    a_route["name"] = brts[i]["name"]
    a_route["routes"] = []
    count = 0
    for j in brts[i]["tripsname"]:
        print("->",j)
        aa_route = dict()
        aa_route["name"] = j
        aa_route["stops"] = []
        aa_route["coords"] = routes[brts[i]["trips"][count]]
        t = trips[brts[i]["tripsid"][count]]
        for k in t:
            print("--->",stops[k]["name"])
            aa_stop = dict()
            aa_stop["name"] = stops[k]["name"]
            aa_stop["coords"] = stops[k]["coords"]
            aa_route["stops"].append(aa_stop)
        a_route["routes"].append(aa_route)
        count = count + 1
    packaged.append(a_route)

for i in packaged:
    print(i["name"])

file_path = 'data.json'
with open(file_path, 'w') as json_file:
    json.dump(packaged, json_file)
```



Finally, the JavaScript script embedded in this [index.html](https://github.com/altilunium/BRT-tj/blob/main/index.html) file will display the information.
