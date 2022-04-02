---
title: "Use Pseudonymized Column as Grouping Key"
date: 2021-12-18T09:22:56+09:00
draft: false
tags: ['Data Integration','Extractor','Extraction','Transformation','Data Job']
---

One of the biggest headache for data engineer like me is how to assure data security when extracting data. Especially personal information should be dealt sensitively, otherwise I may be punished by each region's law (e.g. GDPR).

When I operate Celonis EMS, I try not to extract sensitive information from the beginning, for example I do not extract table of customer address (`ADRC` table in SAP etc.). But this information is sometimes effective for grouping key of counting case etc., so I should mitigate this dilemma. Today I would like to introduce pseudonymization technique to solve this dilemma.

Pseudonymization is processed by hash algorithm, that maps data of an arbitrary size to a bit array of a fixed size (called digest or hash value). The point is converting from original data to hash value is possible but hashed value to original data is quite difficult (not feasible in realistic time). Also if one pair of original data is same value, one pair of hash value is also same. These characteristic is suitable for pseudonymization.

For today's post, first I introduce new table `orders` to store customer, material (ordered item), who created this data (employee) and when it is created. In pgAdmin I execute below SQL to insert four record to `orders`.

```sql
CREATE TABLE orders (
    id BIGINT GENERATED ALWAYS AS IDENTITY,
    PRIMARY KEY(id),
    customer TEXT NOT NULL,
    material TEXT NOT NULL,
	createdBy TEXT NOT NULL,
	createdAt TIMESTAMP NOT NULL
);

INSERT INTO orders (customer,material,createdBy,createdAt) VALUES ('Oda Nobunaga','Gold Plan','Kazuhiko Takata',NOW());
INSERT INTO orders (customer,material,createdBy,createdAt) VALUES ('Toyotomi Hideyoshi','Diamond Plan','Kazuhiko Takata',NOW());
INSERT INTO orders (customer,material,createdBy,createdAt) VALUES ('Tokugawa Ieyasu','Platinum Plan','Kazuhiko Takata',NOW());
INSERT INTO orders (customer,material,createdBy,createdAt) VALUES ('Oda Nobunaga','Platinum Plan','Kazuhiko Takata',NOW());
```

Assuming this `orders` table is located inside my company's local network (on-premise) and I do not want to expose customer name and employee name to outside of my network, but I would like to analyze process of my orders.

When I setup extraction task in Celonis Data Integration, I will click Configure button to set pseudonymized columns (customer and createdBy) as below.

[![image](https://user-images.githubusercontent.com/67397583/146629365-85ea409a-d274-42db-8456-193da230fdd0.png)](https://user-images.githubusercontent.com/67397583/146629365-85ea409a-d274-42db-8456-193da230fdd0.png)

After that I will scroll down to Extraction Preview section and confirm it is not possible to recoginize customer and employee name. Also I can see id 1 and 4 has same value in customer column, these are originally same person. 

[![image](https://user-images.githubusercontent.com/67397583/146629557-be532241-2248-4062-8979-a02b11ddbd09.png)](https://user-images.githubusercontent.com/67397583/146629557-be532241-2248-4062-8979-a02b11ddbd09.png)

If I would like to count orders group by customer, it is possible by pseudonymized customer name. Let's do it in transformation. After extraction has done, execute `SELECT customer,COUNT(*) FROM orders GROUP BY customer;`. Then actually count order by customer has done succesfully.

| customer                                 | COUNT | 
| ---------------------------------------- | ----- | 
| 072643ff50ba49ea26cfcb3ca83978ecdbf74032 | 2     | 
| 3b47fb949d9d0e0a90e281043a7d7271eadd8e52 | 1     | 
| 739782f3b4abc2b1a880c84eab0386f6858c64fe | 1     |

Additionaly I can see how many customers are repeatedly ordered by `SELECT COUNT(*) FROM (SELECT customer,COUNT(*) FROM orders GROUP BY customer) AS a WHERE count > 1;`, not identifying customer name. Grouping key is useful in such case.

Kaz
