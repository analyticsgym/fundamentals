#### LIKE and ILIKE 
- used for matching string patterns
- can combine with other string functions/wildcards to create complex pattern searches
- for text search on large datasets, more advance search methods and/or non-SQL tool might be needed to avoid performance slowdown
- LIKE: is case sensitive 
- ILIKE: is not case sensitive

#### Common wildcards
- '%': matches zero or more characters
- '-': matches any single character
- '[ ]': matches any single character within the specified set or range of characters.

#### Wildcard placement
- '%' placed at start of search looks for ending match
- '%' placed at end of search looks for beginning match
- '%' at start and end used of search used for text contains

#### Simple LIKE and ILIKE examples
- LIKE and ILIKE used within count/case when logic

```
WITH example_tweets(tweet) AS (
VALUES ('Today is such a beautiful day! #sunshine #happy'),
       ('Just got a new iPhone! #Apple #iPhone #technology'),
       ('I love watching movies on weekends. #movies #weekendfun'),
       ('My NEW favorite band is so amazing! #music #band #love'),
       ('Trying out the new pizza place. #food #pizza #delicious'),
       ('Finally got a NEW job! #job #career #success'),
       ('I absolutely love my new workout routine! #fitness #health'),
       ('The latest episode of my favorite show was so intense! #TVshows #drama'),
       ('My cat is so adorable! #cat #pets #cute'),
       ('Attending a great conference this week. #conference #networking #learning')
)

-- For case when logic, when ELSE is not specified, NULL is returned for non-matching cases
SELECT
    -- Count tweets containing the word 'new' with LIKE (case sensitive)
    COUNT(
        CASE 
            WHEN tweet LIKE '%new%' 
            THEN tweet 
        END
    ) AS twt_cnt_contains_new_case_sensitive,
    
    -- Count tweets containing the word 'new' with ILIKE (case insensitive)
    COUNT(
        CASE 
            WHEN tweet ILIKE '%new%' 
            THEN tweet 
        END
    ) AS twt_cnt_contains_new_case_insensitive,
    
    -- Count tweets that start with 'I love' (case insensitive only)
    COUNT(
        CASE 
            WHEN tweet ILIKE 'i love%' 
            THEN tweet 
        END
    ) AS twt_cnt_contains_i_love,
    
    -- Count tweets that include a hashtag
    COUNT(
        CASE 
            WHEN tweet LIKE '%#%' 
            THEN tweet 
        END
    ) AS twt_cnt_includes_hashtag,
    
    -- Count tweets that do not contain 'love' (case insensitive)
    COUNT(
        CASE 
            WHEN tweet NOT ILIKE '%love%' 
            THEN tweet 
        END
    ) AS twt_cnt_does_not_contain_love
FROM example_tweets
```

#### SPLIT_PART 
- takes 3 arguments: input string, delimiter to split string into part, numeric index of part to return

```
WITH sample_urls (id, url) AS (
  VALUES
    (1, 'https://www.example.com/blog/post1'),
    (2, 'https://www.example.com/shop/product1'),
    (3, 'https://www.example.com/gallery/photo1'),
    (4, 'https://www.example.com/services/web-design'),
    (5, 'https://www.example.com/about/team'),
    (6, 'https://www.example.com/blog/post2'),
    (7, 'https://www.example.com/shop/product2'),
    (8, 'https://www.example.com/services/seo'),
    (9, 'https://www.example.com/contact-us'),
    (10, 'https://www.example.com/shop/product3')
)
SELECT
  id,
  SPLIT_PART(url, '/', 3) AS domain,
  SPLIT_PART(url, '/', 4) AS section,
  SPLIT_PART(url, '/', 5) AS sub_section
FROM sample_urls
```

#### UPPER, LOWER
- transform text to uppercase or lowercase
- useful preprocessing before aggregation or to assist string search

```
SELECT
	LOWER('San FRansico') AS lower_city,
	UPPER('los angeles') AS upper_city
```

#### INITCAP
- capitalizes the first letter of each word
- useful for proper noun capitalization

```
WITH ca_example AS (
	SELECT
		UNNEST(
			ARRAY[
				'california',
				'California',
				'CAlifornia',
				'CALIFORNIA'
			]	
		) AS states
)

SELECT INITCAP(states) FROM ca_example
```

#### TRIM
- remove leading, trailing, or leading+trailing patterns from a string

```
WITH store_products AS (
	SELECT
		UNNEST(
			ARRAY[
				'-shoes-',
				' shirts ',
				' pants',
				'pots..'
			]	
		) AS product
)

SELECT
  product AS original_product,
  -- more compact solution possible using REGEXP_REPLACE (see below section notes)
  TRIM(both ' ' from TRIM(both '-' from TRIM(both '.' from product))) AS cleaned_product
FROM store_products
```
#### Regular expressions (regex)
- used for pattern matching and manipulation of strings 
- regex can be used for validation, searching, extraction, replacing, splitting, etc 

#### Regular expressions syntax overview
- commonly used operators in PostgreSQL regex are ~ (matches), !~ (does not match), ~* (case-insensitive match), and !~* (case-insensitive does not match)
- ^ to match the start of a string and $ to match the end of a string
- period(.) is a wildcard that matches any single character except a newline
- square brackets [ ] to define a character class, which matches any single character within the brackets
- [^ ] to create a negated character class, which matches any character not within the brackets
- *, +, ?, {n}, {m,n} to specify how many times a preceding pattern should occur (0 or more, 1 or more, 0 or 1, n times, and between m and n times)
- parentheses ( ) to group patterns together and apply quantifiers to the entire group
- vertical bar | to match one of several alternative patterns
- lookahead and lookbehind assertions: use (?= ) for positive lookahead, (?! ) for negative lookahead, (?<= ) for positive lookbehind, and (?<! ) for negative lookbehind

```
WITH fake_elon_tweets(id, tweet) AS (
    VALUES 
        (1, 'When something is important enough, you do it even if the odds are not in your favor.'),
        (2, 'I could either watch it happen or be a part of it.'),
        (3, 'I think it is possible for ordinary people to choose to be extraordinary.'),
        (4, 'Persistence is very important. You should not give up unless you are forced to give up.'),
        (5, 'The first step is to establish that something is possible; then probability will occur.'),
        (6, 'I’m not trying to be anyone’s savior. I’m just trying to think about the future and not be sad.'),
        (7, 'If the rules are such that you can’t make progress, then you have to fight the rules.'),
        (8, 'Some people don’t like change, but you need to embrace change if the alternative is disaster.'),
        (9, 'It’s OK to have your eggs in one basket as long as you control what happens to that basket.'),
        (10, 'There are really two things that have to occur in order for a new technology to be affordable to the mass market. One is you need economies of scale. The other is you need to iterate on the design. You need to go through a few versions.'),
        (11, 'If you’re trying to create a company, it’s like baking a cake. You have to have all the ingredients in the right proportion.'),
        (12, 'My biggest mistake is probably weighing too much on someone’s talent and not someone’s personality. I think it matters whether someone has a good heart.'),
        (13, 'I always invest my own money in the companies that I create. I don’t believe in the whole thing of just using other people’s money. I don’t think that’s right. I’m not going to ask other people to invest in something if I’m not prepared to do so myself.'),
        (14, 'I wouldn’t say I have a lack of fear. In fact, I’d like my fear emotion to be less because it’s very distracting and fries my nervous system.'),
        (15, 'Life is too short for long-term grudges.'),
        (16, 'I do think there is a lot of potential if you have a compelling product and people are willing to pay a premium for that. I think that is what Apple has shown. You can buy a much cheaper cell phone or laptop, but Apple’s product is so much better than the alternative, and people are willing to pay that premium.'),
        (17, 'I say something, and then it usually happens. Maybe not on schedule, but it usually happens.'),
        (18, 'You want to have a future where you’re expecting things to be better, not one where you’re expecting things to be worse.'),
        (19, 'It is a mistake to hire huge numbers of people to get a complicated job done. Numbers will never compensate for talent in getting the right answer (two people who don’t know something are no better than one), will tend to slow down progress, and will make the task incredibly expensive.'),
        (20, 'A company is a group designed to deliver its output. A company has no purpose in and of itself, but it’s a way that people can work together to achieve something collectively that they couldn’t achieve individually.')
)

-- tweet contains 'company'
-- SELECT tweet FROM fake_elon_tweets WHERE tweet ~ 'company'

-- tweet contains 'ok' case-insensitive (similar to ILIKE)
-- SELECT tweet FROM fake_elon_tweets WHERE tweet ~* 'ok' 

-- tweet starts with 'it is a mistake' case-insensitive (similar to ILIKE)
-- SELECT tweet FROM fake_elon_tweets WHERE tweet ~* '^it is a mistake'  

-- tweet ends with 'grudges' case-insensitive (similar to ILIKE)
-- SELECT tweet FROM fake_elon_tweets WHERE tweet ~* 'grudges.$'  

-- tweets that start with one character then space
-- SELECT tweet FROM fake_elon_tweets WHERE tweet ~ '^. ' 

-- tweets that start vowel
-- SELECT tweet FROM fake_elon_tweets WHERE tweet ~* '^[aeiou]' 

-- tweets contain one or more keywords
-- SELECT tweet FROM fake_elon_tweets WHERE tweet ~* '(future|love|passion|life)'

-- ends with 'myself.' and backslash used to detect literal period
-- SELECT tweet FROM fake_elon_tweets WHERE tweet ~ 'myself\.$'

-- tweet contain people followed by space and 1 or more characters
-- SELECT tweet FROM fake_elon_tweets WHERE tweet ~* 'people .+'

-- tweet does not contain 'I'
-- SELECT tweet FROM fake_elon_tweets WHERE tweet !~ 'I'
```

#### Regex range patterns
- [0-9]: match any number
- [a-z]: match any lowercase
- [A-Z]: match any uppercase
- [A-Za-z0-9]: match any uppercase, lowercase, or number

```
WITH text_data (text_str) AS (
  VALUES
    ('This string has an Amex number: 371234567890123'),
    ('This string has a Visa number: 4012888888881881'),
    ('Amex: 343456789012345'),
    ('MasterCard: 5555555555554444'),
    ('Here is an Amex number: 377894567890123'),
    ('Visa: 4111111111111111'),
    ('Amex again: 340123456789012'),
    ('MasterCard: 5105105105105100'),
    ('And another Amex: 372345678901234'),
    ('And finally, a Visa number: 4222222222222')
)
SELECT
	text_str,
	-- American Express cards start with 34 or 37 and have 15 digits
	-- search for text starting with 34 or 37 and followed by 13 numbers
	text_str ~ '(34|37)[0-9]{13}' AS potential_amex_card
FROM text_data;
```

#### REGEXP_MATCHES
- used to match a regular expression against a string
- returns an array of the substring(s) that match the pattern

```
-- extract phone numbers from text string
WITH comments(comment_id, comment) AS (
    VALUES 
        (1, 'Hi, my number is 123-456-7890'),
        (2, 'Call me at 098-765-4321'),
        (3, 'You can reach me at 111-222-3333'),
        (4, 'My contact is 444-555-6666'),
        (5, '777-888-9999 is where to reach me'),
        (6, 'Do not call me. Please email!')
)

, phone_numbers_extract AS (
SELECT 
	comment_id,
	-- PostgreSQL will drop NULL rows in the regexp_matches result set
	(regexp_matches(comment, '(\d{3}-\d{3}-\d{4})'))[1] AS phone_number
FROM comments
)

SELECT
	c.*,
	-- nulls expected when phone number pattern above doesn't exist in comment
	p.phone_number
FROM comments AS c
LEFT JOIN phone_numbers_extract AS p
	ON p.comment_id = c.comment_id
```

#### REGEXP_REPLACE 
- replaces substrings that match a regular expression

```
-- replace last name with capital letter followed by asterisk(s)
WITH people(first_name, last_name) AS (
    VALUES 
        ('John', 'Doe'),
        ('Jane', 'Smith'),
        ('Alice', 'Johnson'),
        ('Robert', 'Brown'),
        ('Emma', 'Davis'),
	      ('Bob', 'D'),
     	  ('Howard', NULL),
     	  ('Billy', 'San Quin')	
)

SELECT 
    first_name, 
    CASE
			WHEN (LENGTH(last_name) > 1 OR last_name IS NOT NULL)
			THEN UPPER(SUBSTRING(last_name, 1, 1)) || regexp_replace(SUBSTRING(last_name, 2), '.', '*', 'g')
			ELSE last_name
	END AS obfuscated_last_name
FROM people
```

#### REGEXP_SPLTIT_TO_TABLE 
- splits a string using a regular expression as a delimiter, and returns the parts as a new row in a table

```
WITH state_codes AS (
SELECT
	'AL, AK, AZ, AR, CA, CO, CT, DE, FL, GA, 
	HI, ID, IL, IN, IA, KS, KY, LA, ME, MD, 
	MA, MI, MN, MS, MO, MT, NE, NV, NH, NJ, 
	NM, NY, NC, ND, OH, OK, OR, PA, RI, SC, 
	SD, TN, TX, UT, VT, VA, WA, WV, WI, WY' AS state_codes_string
)

SELECT 
	-- splits the string by comma and returns the split part on a new row
	-- 50 row output expected from 1 row input string of states
	regexp_split_to_table(state_codes_string, ',') AS state_code
FROM state_codes
```

#### Concatenate
- join together inputs into a single string output
- typically used with text data type inputs
- however, non-text data that can be converted into a text data type can also be used 

```
-- concat full name and age
WITH people_2(first_name, last_name, age) AS (
    VALUES 
        ('John', 'Doe', 56),
        ('Jane', 'Smith', 67),
	      ('Bob', 'D', 22),
     	  ('Howard', NULL, 19),
     	  ('Billy', 'San Quin', 32)	
)

SELECT
	-- NULL row output expected when 1 or more more of concat inputs is NULL
	first_name || ' ' || last_name || ' | age: ' || age AS full_name_and_age_1,
	-- COALESCE to handle NULL input use case
	first_name || ' ' || COALESCE(last_name, 'No Last Name') || ' | age: ' || age AS full_name_and_age_2
FROM people_2
```

#### STRING_AGG
- concatenate multiple rows of text column into single row
- works like an aggregate math function (can be used to aggregate overall or with group by)

```
WITH video_game_plays(video_game_id, video_game_name, play_timestamp, user_id) AS ( 
    VALUES
      (1, 'Super Mario', TIMESTAMP '2023-01-01 10:00:00', 101),
	    (1, 'Super Mario', TIMESTAMP '2023-02-01 10:00:00', 101),
      (1, 'Super Mario', TIMESTAMP '2023-02-01 10:00:00', 102),
	    (2, 'Zelda', TIMESTAMP '2023-02-01 15:00:00', 102),
      (3, 'Minecraft', TIMESTAMP '2023-03-01 20:00:00', 103),
      (4, 'Fortnite', TIMESTAMP '2023-04-01 21:00:00', 104),
      (5, 'Among Us', TIMESTAMP '2023-05-01 22:00:00', 105),
      (6, 'Call of Duty', TIMESTAMP '2023-01-01 12:00:00', 101),
      (7, 'Cyberpunk 2077', TIMESTAMP '2023-02-01 16:00:00', 102),
      (8, 'FIFA 2023', TIMESTAMP '2023-03-01 18:00:00', 103),
      (9, 'Overwatch', TIMESTAMP '2023-04-01 19:00:00', 104),
      (10, 'Grand Theft Auto V', TIMESTAMP '2023-05-01 23:00:00', 105)
)

SELECT
	user_id,
	-- works similar to Amazon Redshift LISTAGG()
	STRING_AGG(DISTINCT video_game_name,' | ') as video_games_played_unique,
	-- DISTINCT does not work with ORDER BY
	-- different approach needed to string agg distinct videos ordered by first play timestamps
	STRING_AGG(video_game_name,' | ' ORDER BY play_timestamp) as video_games_played_ordered
FROM video_game_plays
GROUP BY 1
```

## Practical Use Cases

#### Text parsing in join logic
- warning this can lead to slow query performance
- consider parsing in upstream logic / prep work (i.e. CTE or temp table to use in join)

#### Regex logic to detect certain email patterns
- detect gmail email addresses that contain lowercase letter, number and a non-alphanumeric character
```
WITH fake_emails AS (
	SELECT
		UNNEST(
			ARRAY[
				'elonmusk@gmail.com',
				'realelonmusk1@spacex.com',
				'oprah_123@gmail.com',
				'jane_smith@gmail.com',
				'skaterchick999@yahoo.com',
				'benfranklin1900@gmail.com',
				'tony_456_xxx@gmail.com'
			]	
		) AS email
)

SELECT
	email
FROM fake_emails
WHERE 1=1
	AND email ~ '@gmail\.com$' 
	AND SPLIT_PART(email, '@', 1) ~ '[a-z]' -- email handle contains lowercase 
	AND SPLIT_PART(email, '@', 1) ~ '[0-9]' -- email handle contains number
	AND SPLIT_PART(email, '@', 1) ~ '[^a-zA-Z0-9]' -- email handle contains any 

-- more compact logic
-- however, this syntax is tough to remember on the fly
-- returns same result as logic above
-- AND SPLIT_PART(email, '@', 1) ~ '^(?=.*[a-z])(?=.*[0-9])(?=.*[^a-zA-Z0-9])'
```

#### Count occurrences in across strings
```
WITH example_tweets(tweet) AS (
VALUES ('Today is such a beautiful day! #sunshine #happy'),
       ('Just got a new iPhone! #Apple #iPhone #technology'),
       ('I love watching movies on weekends. #movies #weekendfun'),
       ('My NEW favorite band is so amazing! #music #band #love'),
       ('Trying out the new pizza place. #food #pizza #delicious'),
       ('Finally got a NEW job! #job #career #success'),
       ('I absolutely love my new workout routine! #fitness #health'),
       ('The latest episode of my favorite show was so intense! #TVshows #drama'),
       ('My cat is so adorable! #cat #pets #cute'),
       ('Attending a great conference this week. #conference #networking #learning')
)

SELECT
	SUM(LENGTH(tweet) - LENGTH(REPLACE(tweet, '#', ''))) as total_hashtags
FROM example_tweets
```