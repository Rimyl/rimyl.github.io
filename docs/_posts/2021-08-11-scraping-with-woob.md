---
layout: post
title:  "Scraping with Woob"
description: Learn what is the scraping framework Woob and how to use it.
tags: [blog, woob, scraping, scrapy, selenium, python, python3, weboob]
lang: en-us, en-uk, en-ca
date:   2021-08-11 11:30:00 +0200
categories: woob
---

The goal of this article is for you to discover what is Woob (previously called Weboob). You are going to learn why I use it, how to install it and the way to use it.

{% toc %}

## Some definitions...

First, we need to put out some meanings to be sure to talk about the same things.
Scraping is an automated way to extract data from websites. You can do it through a script or a program. You may, this way, accumulate data to create a database or just be able to easily consult them.
The downside of this method is that as soon as the website changes, you must update your code to be able to keep retrieving data.

To do so, I use Woob.

## What is Woob?

Woob, Web Outside Of Browser, is a framework made in python giving you all the tools to be able to scrap data across different websites. It is an open-source project that you can find [here](https://gitlab.com/woob/woob).

As a toolbox, it is designed to make the scraping as light as possible. No need to call for or load useless images, pages or javascript.
Woob allows you to handle a lot of different cases:
* navigate through pages made of HTML/CSS
* simulate JavaScript
* keep/simulate cookies
* retrieve JSON
* work with partially malformed XML/HMTL pages
* automate login (even with OTP or virtual keyboard)
* handle APIs REST
* use Selenium if needed even on a part of the website only
* and many other things...

I know what you think right now: "Why use this instead of Scrapy?"
First, the framework starts by giving you more than 200 ready to use websites. More than 200 websites for which you don't need to write code.
There are banks, newspapers, e-shopping, weather broadcasts, video platforms, etc.
The only downside is that the origin of Woob is French, so most websites are French or handled in French. So, it is there, that I encourage you to contribute to this open-source project. ;)

The biggest difference between Woob and Scrapy is their goal  and so the way the retrieval of data is thought.

**Scrapy** is made to crawl. You want a lot of data as fast as possible. You may format them, but it is not a priority.

**Woob** tries to much more mimic the human behavior. It is slower but you have more chance to not be detected as a bot right away and it offers you a much more friendly debug interface. (It is appreciated because of the instability of the scraping.) The framework will only handle formatted data. One of its main goal is really to feed a database.

For this tutorial, I want to know the weather in Dallas, Texas, USA.

## Installation

Let's dirty our hands.
Here, we are going to install Woob on a Debian 10 system.
There is different way to do it. The preferred one is to get the stable version of the framework. The scraping being versatile the more recent the code is, the better.

First, let's install the needed packages:
```shell
$ sudo apt install git python3 python3-pip python3-venv
```

You may need to install optional dependencies if you need to handle pdf files or want to use a graphical application for example.

I will use [pipx](https://pipxproject.github.io/pipx/installation/) to easily handle the python environment. 

Don't forget to add `~/.local/bin` in your PATH environment variable.

```shell
$ pipx install woob
```

I check that everything is ok with Woob and initiate some configuration files.
```shell
$ woob config update
```
This command will also update the modules (see what they are in the next part).

We are now ready to go. \o/

NB: The Woob source code is now in `~/.local/pipx/venvs/woob/lib/python3.7/site-packages/woob`

## Structure

Before using Woob, you need to understand how it is structured.

For now, I will focus on 3 major components.

### Capability

The capability describes the way the data are formatted and what information are retrieved. They are always named *CapSomething*.

For the weather, it is `CapWeather`.
If you have the curiosity and the python knowledge, I encourage you to look at the `woob/woob/capabilities/weather.py` file. This capability is pretty simple and it is a quick way to see how it is structured and what it can do.

### Application

For each capability, there is an application. With it, you can test and use the capability. We will see how in the *Usage* part.
But keep in mind, that it allows you to see the retrieved data in a console in a readable way.

The application linked to `CapWeather` is named `weather` (most of the other ones are named *boosomething*).

### Module

The module (or connector) is THE component that you are going to be the most in contact with.
For each website that you want to exploit there is a module. It is made of 3 main parts that are scattered among 3 corresponding files. (A lot of 3, aren't there?)

**module.py**
For each module, there is always a corresponding `Module` class. It always inherits from one or more capabilities. It explicitly declares what information are retrieved and in which way.
It initiates a browser that you find in the next file.

**browser.py**
All the different kind of browsers that you can find are made to be able to navigate across pages of the selected website.
When on the right page, you can start the scraping itself.

**pages.py**
For each page recognized by the browser, data may be retrieved and send back. It is in this file that you call the scraping methods.

## Usage

Now that you know what capabilities, applications and modules are. I am going to show you how to use those tools.
There are two ways. I advise you to start by using the application so you are able to efficiently test what is happening (and even debug with the right parameters).

The connector I am interested in is named `weather`. It scraps the `weather.com` website.

### Application

```shell
$ woob weather
```
Since it is the first time using this application, you have no data in your backend. Initiating some is easy, because it is a simple application.
![Init backend 1](/assets/2021-08-11-scraping-with-woob-01-init_backend_1.png)

I select the weather connector.
![Init backend 2](/assets/2021-08-11-scraping-with-woob-02-init_backend_2.png)

NB: You will find the backend file at `~/.config/woob/backends`. It is the place where the framework fetches basic and additionnal data for each module. For more complex cases, you may find login/password or which browser to use. You may have several backend entries for each connector.

Now, the loaded backend is the one named *weather* by default.
![Loaded backend](/assets/2021-08-11-scraping-with-woob-03-loaded_backend.png)

With the `help` command, you list all the other ones.
I wanted the weather in Dallas, Texas, USA.

```shell
weather> cities dallas
```

But there are several Dallas in the world. I need to select the right one.
![List cities](/assets/2021-08-11-scraping-with-woob-04-list_cities.png)
You will notice that for each city, there is a number. This number is used as a local identifier. There is a global identifier too that you can display when changing the formatter. For a city, it is made of its coordinates and, in the application, the backend name.
I can now use it to know what is the weather in Dallas.

```shell
weather> current 1
```
![Current Dallas](/assets/2021-08-11-scraping-with-woob-05-current_dallas.png)

You may use the `forecast` command to have the weather for the next days.

NB: If you need the data to be displayed as a table, you need to install an additional python package
```shell
$ pipx inject woob PrettyTable
```
and use the *formatter* parameter using
```shell
$ woob weather -f table
```
or
```shell
weather> formatter table
weather> cities dallas
```
![Cities table](/assets/2021-08-11-scraping-with-woob-06-cities_table.png)

### Script

Now, I am going to get the weather in Dallas through a script. This way, I may do whatever I want with the data.
The framework being in python, so is the script.

First, I need to change the python virtual environment to be able to interact with the framework.
```shell
$ source ~/.local/pipx/venvs/woob/bin/activate
```

```python
from woob.core import Woob

# Initiate a backend entry
w = Woob()
backend = w.build_backend(
    module_name="weather",  # module
    name="weather"  # backend
)

# Fetch cities
cities = [c for c in backend.iter_city_search("dallas")]
for c in cities:
    if "TX" in c.name:
        current = backend.get_current(c.id)  # using the global id of the city
        print(f"{current.date}: {current.temp} - {current.text}")
        break
```
As for the application, I need to initiate a backend for the weather. The `build_backend` method creates a temporary one from the module if the name used does not already exist.
Then I call one of the methods provided by the `Module` of my module to list the cities.
I select the one I am interested in and print the data.

Of course, this code is easily factorizable, but I want it to be as readable as possible.

## Conclusion

Now, you know the basics about Woob.

If you want to know more, you can go to the [official website](https://woob.tech/) and the [documentation](https://dev.woob.tech/contents.html). For any questions, you can ask me or contact the Woob community on #woob on Freenode.

I will probably make another article on the creation of a connector.
Hope it was helpful. (~ ^ u ^)~
