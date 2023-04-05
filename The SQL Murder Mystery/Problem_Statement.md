There's been a Murder in SQL City! '🔪' 
# SQL Murder Mystery

There has been a murder in SQL City.

## Crime Report
```sql
SELECT description FROM crime_scene_report
WHERE type = 'murder' AND city LIKE 'SQL City';
```
yields two witnesses:
> Security footage shows that there were 2 witnesses:
> The first witness lives at the last house on "Northwestern Dr".
> The second witness, named Annabel, lives somewhere on "Franklin Ave".

### Witnesses

1. **Morty Schapiro**

The first witness lives at the **last house** on **Northwestern Dr**.
```sql
SELECT * FROM person
WHERE address_street_name = 'Northwestern Dr'
AND address_number = (
    SELECT MAX(address_number) FROM PERSON
    );
```
**OR**
```sql
SELECT * FROM person 
WHERE address_street_name = 'Northwestern Dr' 
ORDER BY address_number DESC LIMIT 1;
```

2. **Annabel Miller**

The second witness, named **Annabel**, lives somewhere on **Franklin Ave**.
```sql
SELECT * FROM person
WHERE address_street_name = 'Franklin Ave'
AND name LIKE 'Annabel %';
```

#### Morty Schapiro
```sql
SELECT person.name, interview.transcript
FROM interview
INNER JOIN person
ON person.id = interview.person_id
WHERE person.name = 'Morty Schapiro';
```

**Transcript:**
> I heard a gunshot and then saw a man run out. He had a "Get Fit Now Gym" bag.
> The membership number on the bag started with "48Z". Only gold members have those bags.
> The man got into a car with a plate that included "H42W". 


#### Annabel Miller
```sql
SELECT person.name, interview.transcript
FROM interview
INNER JOIN person
ON person.id = interview.person_id
WHERE person.name = 'Annabel Miller';
```

**Transcript:**
> I saw the murder happen, and I recognized the killer from my gym when I was working out last week on January the 9th.

### Murderer
* *Get Fit Now* member
* gold member
* membership no. starts with: `48Z`
* worked out on January, 9th
* car plate included: `H42W`

#### Gym
```sql
SELECT person_id FROM get_fit_now_member
WHERE membership_status = 'gold'
AND id LIKE '48Z%'
AND id IN (
    SELECT membership_id FROM get_fit_now_check_in
    WHERE check_in_date = '20180109'
    );
```

| id    | person\_id | name          | membership\_start\_date | membership\_status |
|-------|-----------|---------------|-----------------------|-------------------|
| 48Z7A | 28819     | Joe Germuska  | 20160305              | gold              |
| 48Z55 | 67318     | Jeremy Bowers | 20160101              | gold              |

```sql
SELECT * FROM person
WHERE id IN (
    SELECT person_id FROM get_fit_now_member
    WHERE membership_status = 'gold'
    AND id LIKE '48Z%'
    AND id IN (
      SELECT membership_id FROM get_fit_now_check_in
      WHERE check_in_date = '20180109'
      )
    );
```

| id    | name          | license\_id | address\_number | address\_street\_name   | ssn       |
|-------|---------------|------------|----------------|-----------------------|-----------|
| 28819 | Joe Germuska  | 173289     | 111            | Fisk Rd               | 138909730 |
| 67318 | Jeremy Bowers | 423327     | 530            | Washington Pl, Apt 3A | 871539279 |

#### Car
```sql
SELECT * FROM drivers_license
WHERE gender = 'male'
AND plate_number LIKE '%H42W%';
```

| id     | age | height | eye\_color | hair\_color | gender | plate\_number | car\_make  | car\_model |
|--------|-----|--------|-----------|------------|--------|--------------|-----------|-----------|
| 423327 | 30  | 70     | brown     | brown      | male   | 0H42W2       | Chevrolet | Spark LS  |
| 664760 | 21  | 71     | black     | black      | male   | 4H42WR       | Nissan    | Altima    |



### Putting it together
```sql
SELECT * FROM person
WHERE id IN (
    SELECT person_id FROM get_fit_now_member
    WHERE membership_status = 'gold'
    AND id LIKE '48Z%'
    AND id IN (
      SELECT membership_id FROM get_fit_now_check_in
      WHERE check_in_date = '20180109'
      )
    )
AND license_id IN (
    SELECT id FROM drivers_license
    WHERE gender = 'male'
    AND plate_number LIKE '%H42W%'
    );
```

**The murderer is Jeremy Bowers.**

Message from the Makers:
> Congrats, you found the murderer! But wait, there's more...
> If you think you're up for a challenge, try querying the interview transcript of the murderer to find the real villian behind this crime.
> If you feel especially confident in your SQL skills, try to complete this final step with no more than 2 queries.
> Use this same INSERT statement to check your answer.


## Part 2
```sql
SELECT transcript FROM interview
JOIN person
ON person.id = interview.person_id
WHERE person.name = 'Jeremy Bowers';
```
> I was hired by a woman with a lot of money.
> I don't know her name but I know she's around 5'5" (65") or 5'7" (67").
> She has red hair and she drives a Tesla Model S.
> I know that she attended the SQL Symphony Concert 3 times in December 2017. 


### Concert
```sql
SELECT person.* FROM facebook_event_checkin
JOIN person
ON person.id = facebook_event_checkin.person_id
WHERE event_name = 'SQL Symphony Concert'
AND date LIKE '201712%'
GROUP BY person_id
HAVING COUNT(*) = 3;
```

| id    | name             | license\_id | address\_number | address\_street\_name | ssn       |
|-------|------------------|------------|----------------|---------------------|-----------|
| 24556 | Bryan Pardo      | 101191     | 703            | Machine Ln          | 816663882 |
| 99716 | Miranda Priestly | 202298     | 1883           | Golden Ave          | 987756388 |

### Add Car
```sql
SELECT person.* FROM facebook_event_checkin
JOIN person
ON person.id = facebook_event_checkin.person_id
JOIN drivers_license
ON person.license_id = drivers_license.id
WHERE event_name = 'SQL Symphony Concert'
AND date LIKE '201712%'
GROUP BY person_id
HAVING COUNT(*) = 3;
```

| id    | name             | license\_id | address\_number | address\_street\_name | ssn       |
|-------|------------------|------------|----------------|---------------------|-----------|
| 99716 | Miranda Priestly | 202298     | 1883           | Golden Ave          | 987756388 |



> Everyone in SQL City hails you as the greatest SQL detective of all time.

