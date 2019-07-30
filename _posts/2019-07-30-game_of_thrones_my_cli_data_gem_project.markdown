---
layout: post
title:      "Game of Thrones: My CLI Data Gem Project"
date:       2019-07-30 14:02:29 +0000
permalink:  game_of_thrones_my_cli_data_gem_project
---


For the final Ruby portfolio project I wanted to do something lighthearted, but still a little special.  I had read a number of other blogs of some previous projects and had seen a couple of my classmates' working projects during study groups and I new fairly quickly that I wanted to make something that didn't take itself too seriously.  I combined that intention with my background in residential construction and the possible themes for the project coalesced fairly quickly around the inevitable:  I had to make a CLI game about toilets , and I had to name it Game Of Thrones (queue the intro music!)

The gameplay for this program is very simple.  Users choose which category of toilet they want, and the game presents them with three random toilets from that category.  The user then guesses which toilet is the most expensive.  There is also an option to look at the total available data (including price - no cheating!) for any of the three toilet guesses.

The scraping for this program is done on the website of Kohler International - a plumbing company that is the classic all-American success story of midwestern industrialism and family entrepeneurship (The Kohler family are now 4th Generation owners of the business, with now global reach in its sales and marketing).  Their website is beautiful, as are their toilets.  The structure of the website was fairly easy to manipulate for the project.  Toilets are presented in their own dedicated section with options to choose from one of four different style categories (2 piece, wall-hung, etc...).  Choosing a category then took you into the catalogue of corresponding toilet models.  Some categories had just a few models in their catalogue (ie intelligent toilets), while others clearly have 25+ (ie two-piece).  Both tiers of information - the different categories and then their corresponding toilet catalogues - were scrapable for the project.  Unfortunately the data for the individual toilets were loaded with Javascript and evaded my scraping efforts (for now...).


Once I new how the relationship between the categories and the toilets behaved I was able to sketch out a rough idea for how the program would work.  To keep with best practice,  classes for the categories and the toilets would be responsible only for producing objects as needed in the program, so the abstract 'controller' class would have to be the main workhorse of the game. Further additions to the code to ensure that unnecessary scraping was avoided, and that the CLI was typo-proof, kept the program neatly contained within itself and the possible objects that could possibley be generated.  


The toilet categories and the corresponding catalogues clearly presented a comparable "has many" and "belongs to" object relationship, respectively.  It made sense to put together an object instantiation technique that wired up their relationships as the objects were generated.  Both the toilet and category objects contain attributes that self populate each other ⁠— either as arguments that are passed through to the initialize method, or as an array that is populated automatically ⁠— through instantiation.

Formatting strings pulled from the scrape into usable data elements was also necessary for this game.  For example, toilet pricing presented on the website came in the format of $1,234.56, so generating toilet objects for the program also meant generating useable (aka sortable) price values for the program.  Trial and error quickly produced an efficient technique for controlling how user input was captured and measured against both the hard-coded parameters of the program, as well as the variable content that was scraped off the internet.

In all, I found the project exercise to be deeply rewarding in my coding studies.  I needed to do 2 major refactorings of the program after the game was operable - once to remove unnecessary hashes, and the other to consolidate and optimize the class file structures - but even through those refactorings I enjoyed the experience of getting more intimate with the methods and how they collaborated the objects across classes.

Completing the CLI Data Gem project confirmed for me that I truly enjoy coding.  It is something that I am now looking forward to being really, really, really good at.
