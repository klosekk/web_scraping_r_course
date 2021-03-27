---
title: "Web Scraping with rvest"
author: "Kamil Klosek"
date: "3/27/2021"
output:
  html_document:
    keep_md: true
---



## Introduction

In this module, we are going to learn how to harness HTML source code for web scraping using the rvest pacakge. Therefore, knowledge in HTML/XML and CSS are highly useful for this module. Our goals are:  
  
1. Estimate size of the workload
2. Automatize data collection process
3. Archive raw data
4. Extract and format data

Alternatively, some web services allow us to download data using APIs. This will be the content of the next module.

## Legal side of web scraping

There are at least two points to consider before engaging in web scraping:

1. Make sure that it is permissible to download specific content 
- Read /robots.txt file
- Read Terms of Agreement if you extract data from a password-protected site
2. Minimize your web traffic
- Before considering web scraping, check whether the website provides a database or API (use those first!)
- Implement reasonable request delays (might be mentioned in robots.txt)
- Consider scraping the website in off-peak hours (e.g. at night)
3. Identify yourself through a User Agent

Every website (should) have a robots.txt file, accessible through root-homepage-name/robots.txt in the URL-field in your browser, which specifies which directories are Allowed/Disallowed to scrape from.

Be mindful that web scraping creates traffic. If too many people engage in web scraping at the same time, you can inadvertently make it more difficult for human users to access the website. In worst case, you might crash the hosting server. If you are lucky, nothings happens, otherwise your IP address might be blocked (which would be unfortunate if you access websites from an institution VPN), and in worst case you will get a letter from a lawyer demanding compensation for the damage you created. Therefore, behave in an appropriate and mindful manner when you engage in web scraping.

If you web scrape inside the European Union, consider reading the [directive 2019/790](https://eur-lex.europa.eu/eli/dir/2019/790/oj) on on copyright and related rights enacted by the European Parliament and Eurpean Council.

## HTML/XML syntax

HTML stand for **H**yper **T**ext **M**arkup **L**anguage and is an offshoot of XML which stands for **Ex**tensible **M**arkup **L**anguage. HTML forms the backbone of the majority of sites on the web, therefore it is curcial to understand its fundamental structure from the perspective of a web scraper.

In markup language, plain text is categorized using tags with **<>** angle brackets. For instance, in HTML the tag \<title\> tells your browser to display *Title Name* as title: \<title\>Title Name\<\/title\>. Together the combination of tags and text is referred to as *element*. While the syntax is the same, the meaning tags in HTML are strictly defined by the [World Wide Web Consortium (W3C)](https://www.w3schools.com/html/), whereas XML allows the user to create own tags. Another important feature for us are **attributes**. Attributes can be attached to tags, e.g. \<a href="URL"\>Text\</a\>. In this example, we assign a URL to text embedded in the \<a\>\</a\> tag. 
  
Another important feature of HTML/XML is their tree-structure. Such tree-structured text is also called **Document Object Model (DOM)**. Tags like \<body\>\</body\> can include multiple tags, e.g. \<title\>, \<p\>, or \<ul\> to name a few. This nested form allows us web scrapers to pinpoint at those tags which as closely as possible define the information we want to extract. For instance, in a structure such as \<body\>\<p\>Text\<ul\>Text\</ul\>\</p\>\</body\> we might want to extract information provided as unstructured list (ul) by singling out **ul** as our target tag. Another way of thinking about tags in a tree-structure is to view them as nodes that can split into several additional nodes.

Lastly, HTML source code can contain **CSS (Cascading Style Sheets)** which is used to format the layout of websites. Website developers define tags with CSS information, e.g. p.highlight where p stands for the paragraph tag \<p\> and *highlight* represents a specific layout style provided in curly brackets: p.highlight {color : green; font-size:150%}. The source code can then contain the CSS information using the class attribute: \<p class = "highlight"\>Text\</p\>. One can also define the CSS style directly within a tag, but this becomes tedious and is less readable, especially when many different styles are used.

## XPATH

**XPATH** stands for **X**ML **P**ath **L**anguage and is a web technology that allows the user to extract information from HTML/XML documents. Its structure is hierarchical meaing that we move alongside tags in the tree from the highest-order tags (e.g. \/html\/body) to lower-order tags (e.g. \/div\/p\/ul). While we can use absolute paths (\/html\/body\/div\/p\/ul), we might also use relative paths using \/\/ (e.g. \/\/p\/ul). This means that we extract information from all \<ul\> which are a node below all \<p\>. We however disgregard those \<ul\> which are not connected with a \<p\>.

## How to identify the information I need in the HTML source code

There is no water-proof way how to find exactly those tags than only contain that information that you want and nothing more. However, there are tools and ways how to facilitate the process.

The most useful tool is [SelectorGadget](https://chrome.google.com/webstore/detail/selectorgadget/mhjhnkcfbdhnjickkkdbjoemdmbfginb) which is available as Chrome extension. Selector gadget allows you to point-and-click at elements of a website that contain your desired information. Make sure that only information is highlighted in green/yellow that contains what you want to extract. In best circumstances, you will obtain a tag or a combination of tags that we can use later as nodes to separate irrelevant from relevant information.

Should you fail to obtain some straightforward combination of tags, the second option is to use **XPATH**. You can use SelectorGadget to obtain the XPATH or you right-click on your desired information and select *inspect* (on Chrome). Evaluate the highlighted source code and see whether you can find a pattern that connects the data you are looking for with a combination of tags. For instance, you might be interested in obtaining the titles of articles and you discover that those titles you are searching for all use class="media_tag2" within an \<a\> tag. With this information at hand, you can use XPATH to directly access all text with \<a class="media_tag2"\>.

## Using HTML for web scraping in R

In R, we typically engage in the following process:

1. Download the HTML source code
2. Use parsing to create a DOM
3. Harness the tree-structure of your DOM to identify elements with relevant information
4. Use R functions to extract information
5. Create data objects such as lists or data frames suitable for data analysis









## Credits

The book *Automated Data Collection with R: A Practical Guide to Web Scraping and Text Mining* by Simon Munzert, Christian Rubba, Peter Meibner (2015) was of great help to understand websites from the perspective of web scraping.  
The author of this module also benefited immensely from the course *Big Data Collection and Management in R* provided by Dr. Matt W. Loftis at the ECPR Methods School in 2021.

