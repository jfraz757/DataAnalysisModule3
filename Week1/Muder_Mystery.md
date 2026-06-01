# Murder Mystery Query Log

## Starting Notes:
- Murder occured on 01/15/2018 in SQL City.
- Get the crime scene report from the Police Database. 

### Step 1: Look at the Crime Scene Report.
<mark>SELECT *
FROM crime_scene_report</mark>

- This command gave me a full view of the Crime Scene Report
### Step 2: Narrow down the Crime Scene Report to only show incidents that occured on 01/15/2018 in SQL City.
<mark>SELECT *
FROM crime_scene_report
WHERE date = 20180115 and city = "SQL City"</mark>

- This query narrowed down the resules to 3 incidents. Only one of which is a murder. 
- The description of the incident is as follows:
    - "Security footage shows that there were 2 witnesses. The first witness lives at the last house on "Northwestern Dr". The second witness, named Annabel, lives somewhere on "Franklin Ave"."

### Step 3: Investigate the witnesses.
- Witness #1 : Last house on Northwestern Dr.
    - I'm opening up the "person databse to see if it shows addresses:
        - <mark>Select * 
            FROM Person</mark>
        - Now that I know addresses are included in "Person" I'm going to narrow it down further to people who live on "Northwestern Drive and the adress number is the largest.    
            - **Note:** I struggled with this one a little. At first I put ORDER BY on the same line as WHERE, which gave a syntax error. Then I tried putting ORDER BY before WHERE on a separate line. This also gave a syntax error. Finally, I tried ORDER BY on a line under WHERE, and it worked!
        - <mark>Select * 
            FROM Person
            WHERE address_street_name = "Northwestern Dr"
            ORDER BY address_number desc </mark>
            - This resulted in the person:
                - id	name	license_id	address_number	address_street_name	ssn
                - 14887	Morty Schapiro	118009	4919	Northwestern Dr	111564949

- Witness #2: Named "Annabel", lives somwhere on "Franklin Ave"
    - Using the "Person" database, I'll now filter the results for people named "Annabel" that live somewhere on "Franklin Ave".
        - **Note:** First I tried name = Annabel and got no results. Then I just looked at the whole table and realized I would have to use a 'starts with' command in order to get results for Annabel, because her last name is unknown.
    - <mark>Select * 
        FROM person
        WHERE address_street_name = "Franklin Ave" 
        AND name Like 'Annabel%'</mark>
        - This query resulted in the following:
            - id	name	license_id	address_number	address_street_name	ssn
            - 16371	Annabel Miller	490173	103	Franklin Ave	318771143

### Step 4: Check out the Interviews for the two Suspects. 
- Interview 1: Morty Schapiro
    - <mark>Select * 
        FROM interview
        WHERE person_id = 14887</mark>
        - "	I heard a gunshot and then saw a man run out. He had a "Get Fit Now Gym" bag. The membership number on the bag started with "48Z". Only gold members have those bags. The man got into a car with a plate that included "H42W".
- Interview 2: Annabel Miller
    - <mark>Select * 
        FROM interview
        WHERE person_id = 16371</mark>
        - "I saw the murder happen, and I recognized the killer from my gym when I was working out last week on January the 9th."

### Step 5: Investigate the Leads from the Interviews.
#### Lead 1 Investigation:
    - "Get Fit Now Gym"
    - Membership number that starts with 48Z. 
    - Gold Member
    - Liscence plate that includes H42w2
- <mark>Select * 
    FROM get_fit_now_member
    WHERE membership_status = "gold" 
    AND id Like '%48Z%' </mark>
    - This search returned two members:
        - id	person_id	name	membership_start_date	membership_status
        - 48Z7A	28819	Joe Germuska	20160305	gold
        - 48Z55	67318	Jeremy Bowers	20160101	gold
#### Lead 2 Investigation:
    - Killer was at the gym on 01/09/2018
- <mark>Select * 
    FROM get_fit_now_check_in
    WHERE check_in_date = 20180109 and membership_id = ("48Z7A" OR "48Z55")</mark>
    
    **Note:** This was my first query and it returned no reults
- <mark>Select * 
    FROM get_fit_now_check_in
    WHERE check_in_date = 20180109 and membership_id = "48Z7A" OR "48Z55"</mark>

    **Note:** This second attempt gave me a full table with all of the membership_ids. 
- <mark>Select * 
    FROM get_fit_now_check_in
    WHERE check_in_date = 20180109 and membership_id = "48Z7A"</mark>

    <mark>Select * 
    FROM get_fit_now_check_in
    WHERE check_in_date = 20180109 and membership_id = "48Z55"</mark>
    
    **Note:** I ran these two commands sepraretly and confrimed that both of them are in the "get_fit_now_check_in" database on the same date. But I want them to show in the same table. 
- <mark>Select * 
    FROM get_fit_now_check_in
    WHERE check_in_date = 20180109 
    AND (membership_id = "48Z7A" OR membership_id = "48Z55") </mark>

    **Note:** I used Gemini to troubleshoot why my script wasn't returning two rows. This is the format that I needed.
    -   This search gave me the same two people back and their check in/out times. It doesn't seem to be helpful for now. 

### Step 6: Check out License Plates
The Step 5 investigation gave me two new names to investigate, but more importantly a license plate number I can look into includes "H42w2".
- <mark>SELECT * 
    FROM drivers_license
    WHERE plate_number LIKE '%H42w2%'</mark>
    - This query yeilded one result:
    - id	age	height	eye_color	hair_color	gender	plate_number	car_make	car_model
    - 423327	30	70	brown	brown	male	0H42W2	Chevrolet	Spark LS

Now I need to see which of the two suspects has the id number: 4223327

### Step 7: Join Tables by ID Number

I joined the Get Fit Now Member and Check In Tables togther by Membner ID so I could have all of their information in one place.
- <mark> SELECT * 
    FROM get_fit_now_member
    JOIN get_fit_now_check_in ON get_fit_now_member.id = get_fit_now_check_in.membership_id
    Where id IN ("48Z7A", "48Z55")</mark>
    - id	person_id	name	membership_start_date	membership_status	membership_id	check_in_date	check_in_time	check_out_time
    - 48Z55	67318	Jeremy Bowers	20160101	gold	48Z55	20180109	1530	1700
    - 48Z7A	28819	Joe Germuska	20160305	gold	48Z7A	20180109	1600	1730

### Step 8: Confusion.

I have been tripped up here. The ID in the Driver's Liscence Table to not match and of the IDs in my Get Fit Now Member or Check In Tables. I need to try something different. 

### Step 9: Try the Person Table

I decided to look at the Person table and see if my two suspects names where listed.

- <mark>SELECT * 
    FROM person
    WHERE name IN ("Jeremy Bowers", "Joe Germuska")</mark>
    - id	name	license_id	address_number	address_street_name	ssn
    - 28819	Joe Germuska	173289	111	Fisk Rd	138909730
    - 67318	Jeremy Bowers	423327	530	Washington Pl, Apt 3A	871539279
### Step 10: Use the liscense_id from the Person Table to narrow down the suspect by matching liscinse id and plate number. 

I joined Driver License and Person on the License column and filtered the results by the license plate given in Setp 6. 
- <mark>SELECT * 
    FROM drivers_license
    JOIN person ON drivers_license.id = person.license_id
    WHERE plate_number LIKE '%H42w2%' </mark>
    - id	age	height	eye_color	hair_color	gender	plate_number	car_make	car_model	id	name	license_id	address_number	address_street_name	ssn
423327	30	70	brown	brown	male	0H42W2	Chevrolet	Spark LS	67318	Jeremy Bowers	423327	530	Washington Pl, Apt 3A	871539279

### Step 11: Check my Solution.
Alright.  It's time to see if Jeremy Bowers is the murderer.
- <mark> INSERT INTO solution VALUES (1, 'Jeremy Bowers'); SELECT value FROM solution;</mark>
    - "Congrats, you found the murderer! But wait, there's more... If you think you're up for a challenge, try querying the interview transcript of the murderer to find the real villain behind this crime. If you feel especially confident in your SQL skills, try to complete this final step with no more than 2 queries. Use this same INSERT statement with your new suspect to check your answer."

### Step 12: Fine, I'll Keep Going.
- Find the murderer's transcript:
    - <mark>SELECT * 
    FROM interview
    WHERE person_id = 67318</mark>
        - "I was hired by a woman with a lot of money. I don't know her name but I know she's around 5'5" (65") or 5'7" (67"). She has red hair and she drives a Tesla Model S. I know that she attended the SQL Symphony Concert 3 times in December 2017."
### Step 13: Back to the Drivers License Table.
- <mark>SELECT * 
    FROM drivers_license
    WHERE height IN (65, 67) AND gender = "female"
    AND hair_color = "red" AND car_make = "Tesla"</mark>
    - id	age	height	eye_color	hair_color	gender	plate_number	car_make	car_model
    - 918773	48	65	black	red	female	917UU3	Tesla	Model S
### Step 14: Run the License Number in the Person Table.
- <mark>SELECT * 
    FROM person
    WHERE license_id = 918773</mark>
    - Red Korb
## Well hot damn! I was wrong. Apparently it wasn't Red Korb that put out the hit, now I need to go back and see what went wrong. 
### Step 15: Back to the Drivers Licnese Table
My original query in Step 13 was too strict on the height requirement. I looked for the two hieghts given specifically. Not the considering the values in between. I've taken hieght out of the query in Step 13, and returned these two results (minus the one we know is innocent):
- 202298	68	66	green	red	female	500123	Tesla	Model S
- 291182	65	66	blue	red	female	08CM64	Tesla	Model S

### Step 16: Check out the Facebook Event Check In.
- <mark>SELECT * 
    FROM facebook_event_checkin
    WHERE event_name = 'SQL Symphony Concert' AND date = 20171203</mark>
    - person_id	event_id	event_name	date
    - 79312	1143	SQL Symphony Concert	20171203

### Step 17: Identify the Scuspect by their Person_ID
- <mark>SELECT * 
    FROM person
    where id = 79312</mark>
    - Randell Staheli

### Step 18: Confirm it was Randell Staheli.
## I WAS WRONG AGAIN!

### Step 19: Return to the Facebook Event Check In Table.
This time my script will look at the SQL Symphony Concert for dates in 12/2017 with a person id that appeared 3 times and is not Red Korb who we know is innocent. 
- <mark>SELECT * 
    FROM facebook_event_checkin
    WHERE event_name = 'SQL Symphony Concert' AND date Like '201712%' 
    AND person_id
    GROUP BY person_id
    HAVING COUNT(person_id) = 3</mark>
    - 24556	1143	SQL Symphony Concert	20171224

### Step 20: Identify the Suspect by their Person_ID and Confirm it was them. 
- <mark> SELECT * 
    FROM person
    where id = 99716</mark>
    - Mirands Priestly
## Hooray!






