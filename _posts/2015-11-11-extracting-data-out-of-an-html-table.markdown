---
layout: post
title: "Generating SQL from out of an HTML table"
date: 2015-11-11
categories: [SQL]
keywords: "Google Docs, HTML, SQL"
description: "Generating SQL from out of an HTML table"
---

So I learned a new trick today that I want to share! 

####The Scenario

I have a database table holding infromation about countries, including the 2 letter country codes as specified by ISO 3166-1.
Now, I required to add another column for 3 letter country codes.

For example: **Australia &#10141; AU &#10141; AUS**

There are roughly **190+ countries** in the table that each required SQL script to update them.

####Solution

1. First I found a page that listed all the countries with their respective 2 and 3 letter codes incidentally on [wikipedia](https://en.wikipedia.org/wiki/ISO_3166-1).

2. Next, I needed to extract the data from the table and write out some SQL scripts.

    Here came the challenge. A few ideas came to mind:

    - **Regex**: God help me, but I'm not one of those creatures blessed with the ability to do much regex, though I've met a few in the wild.
    - **HTML Agility Pack**: While I'm proficient in C#, I really didn't want to have to File &#10141; New just to get this task done.
    - **Hand write the SQL**: Probably not more than a 5 minute job, but how tedious!

    This is where I came across this little gem: **Google Sheets** and specifically a little command called **[ImportHtml](https://support.google.com/docs/answer/3093339?hl=en)**.

3. Open Google Docs and go to Google Sheets (you will need to login with an google account - psst: It's free!).

4. Create a new worksheet.

5. Go to top most left cell (A1) and type in **=IMPORTHTML(**. It should give you a hint like in the screenshow below.

    <div class="centered">
        [<img src="/images/google-sheets.png"  alt="Google Sheets" style="width: 640px; height: 379px"/>](/images/google-sheets.png)
    </div>

6. The command requires you to pass in the following parameters:
    - **URL**: The URL you wish to import from.
    - **Element type**: either "table" or "list"
    - **Index**: which table or list you wish to import off the page, with an index starting from 1.
 
    My command looked something like this: **=IMPORTHTML("https://en.wikipedia.org/wiki/ISO_3166-1", "table", 1)**
 
7. Voila. Magic. The data appears like so:
 
    <div class="centered">
        [<img src="/images/google-sheets-with-data.png"  alt="Google Sheets with data" style="width: 640px; height: 379px"/>](/images/google-sheets-with-data.png)
    </div>

8. Now, I can create my SQL statement by referencing the cells and using some commands to concatenate the data.

    - My SQL statement should look like: **`UPDATE COUNTRY SET CountryCode3Letter = 'AUS' WHERE CountryCode2Letter = 'AU'`**
    - Use the **Concatenate** command to build it up in the sheet.
    - For example, in my case it was: **`=CONCATENATE("UPDATE COUNTRY SET CountryCode3Letter = '", C3, "' WHERE CountryCode2Letter = '", B3, "'")`**
    
    <div class="centered">
        [<img src="/images/google-sheets-create-query.png"  alt="Create query" style="width: 640px; height: 379px"/>](/images/google-sheets-create-query.png)
    </div>      

9. Finally, I copy that cell and paste it for all countries in the sheet.

    <div class="centered">
        [<img src="/images/google-sheets-copy-query.png"  alt="Copy query" style="width: 640px; height: 379px"/>](/images/google-sheets-copy-query.png)
    </div>
    
    <div class="centered">
        [<img src="/images/google-sheets-done.png"  alt="All queries generated" style="width: 640px; height: 379px"/>](/images/google-sheets-done.png)
    </div>

10. Too easy - Job done. Next!

  
