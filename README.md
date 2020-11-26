# Frontend Applications

## 📚 Table of contents

- [🤔 About](#-About)
  - [🛠 Build with](#-Build-with)
- [🔍 Research question](#-Research-question)
- [🔧 Installing the project](#-Installing-and-using-the-project)
  - [🚀 Launch the project](#-Launch-the-project)
  - [✏ Linting](#-Linting-the-project)
- [📝 Sources](#-Sources)
- [🗺️ License](#%EF%B8%8F-license)

## 🤔 About

![Screenshot data vizualization](https://raw.githubusercontent.com/wiki/Vuurvos1/frontend-data/img/appScreenshot.png)

For the Volkskrant, we will be research/explore several datasets about a topic that journalist might write an article about. In this case, it is all about parking.

For this article visualizations will be made using data from RDW, these datasets contain information about car parking and which vehicles park where inside the Netherlands. The data from these datasets will be visualized using the D3 JavaScript library.

Check out the [wiki](https://github.com/Vuurvos1/frontend-applications/wiki) of this repository to find out more detailed information about the project.
Also, make sure to check my [Frontend Data](https://github.com/Vuurvos1/frontend-data) and [Functional Programming](https://github.com/Vuurvos1/functional-programming) repository for some of my progress

### See it live

The site is also hosted on Surge, check it out [here](https://sam-fa.surge.sh/)

### 🛠 Build with

- Node.js
- D3.js
- Svelte
- NPM packages

## 🔍 Research question

Where do I have the most chance to find a parking location?

- Where are the most parking spots

  - Calculate the number of parking spots per x amount of residents

- How accessible are the parking spots?

  - Get a list if the parking garage has disabled access

- How much do I have to pay to park?

  - Get pricing per hour for certain garages

- What about parking during holidays and on the weekends (like Sunday)
  - See if there is data available for parking availability during certain days

**Cool to have**

- How full are parking garages/spots currently
  - Find an API or multiple to see how many cars are parked wherein real-time

### Datasets needed

- Location of parking spots
  [GEO-Parkeer-Garages](https://opendata.rdw.nl/Parkeren/GEO-Parkeer-Garages/t5pc-eb34) (Parking garages locations)
- Where are the most parking spots?
  [Open-Data-Parkeren-SPECIFICATIES-PARKEERGEBIED](https://opendata.rdw.nl/Parkeren/Open-Data-Parkeren-SPECIFICATIES-PARKEERGEBIED/b3us-f26s) (capacity of parking garages)
- How many parking spots are filled (preferably realtime)

When combining these datasets based on areaIds and no match isfound, I will be voiding these values since I don't have the information needed to plot them properly.

### Concept sketch

My concept is to create a map showing the parking density inside the Netherlands, these values will be normalized with the population density to prevent a visualization that is almost the same as the population density in the Netherlands

![Map](https://raw.githubusercontent.com/wiki/Vuurvos1/frontend-data/img/datavizSketch.png)

**Tooltip for extra info**

![Tooltip](https://raw.githubusercontent.com/wiki/Vuurvos1/frontend-data/img/datavizSketchTooltip.png)

### Assumptions

There is less chance to find a parking spot in the middle of a city than further away from the city centre.
You have a higher chance to find a parking spot throughout the week than at the weekend.

## 🔧 Installing and using the project

First of all, make sure you have **Node.js**, **NPM** and **Git** installed

1. Choose or make a new directory to clone the project to
2. Clone the repository
   `git@github.com:Vuurvos1/frontend-applications.git`
3. Cd into the project folder
4. Run `npm install` to install the needed npm packages
5. Run `npm run build` to build all needed files

### 🚀 Launch the project

You can start the project using `npm start`
or run `npm run dev` if you are a developer

By default, the project will be hosted on [localhost:5000](http://localhost:5000)

## 📝 Sources

[Chubby Racoon 🦝](https://github.com/rowinruizendaal) for brainstorming code and other ideas

## 🗺️ License

Author: [Vuurvos1](https://github.com/Vuurvos1), license by [MIT](https://github.com/Vuurvos1/frontend-applications/blob/main/LICENSE)
