---
layout: page
title: Infection Simulator project
permalink: /infection-simulator-project/
---

This project is an interactive web application that visually simulates the spread of an infection through a community. Developed using JavaScript, it is showcased on a simple [web interface](https://infection-simulator-953aec947747.herokuapp.com/), highlighting the dynamic aspects of infection spread. The backend, powered by Python (Flask), manages the application's server-side operations. It is hosted on Heroku, leveraging cloud capabilities for accessibility. Continuous Integration and Continuous Deployment (CI/CD) practices are integrated via GitHub Actions, ensuring streamlined updates and maintenance of the application. 

To see more of the code running the app, check out the [GitHub repository](https://github.com/reedhodges/infection_simulator).

### Explanation of the simulation

The simulation uses a simple model for the spread of an infection through a community.  Starting with one infected individual, at each time step there is a chance that a healthy individual will be infected by a sick individual nearby.  Infected individuals will either recover or die after a set amount of time.  The spread is affected by parameters such as infection chance, mortality rate, inoculation rate, and inoculation efficacy.  The parameters can be adjusted by the user to see their effect on the simulation.   

At the initial launch of the web app, the parameters are set to their default values.

```js
let grid_size = 50;
let num_dots = 250;
let infection_chance = 0.5;
let recovery_time = 200;
let infection_distance = 15;
let frame_rate = 30;
let mortality_rate = 0.2;
let inoculation_rate = 0.1;
let inoculation_efficacy = 0.5;
```

Each individual in the simulation is an instance of the `Dot` class. The various properties of each dot are initialized (e.g., position, inoculation status).  The color of the dot is calculated by the `getColor()` function.  The functions `updatePosition()` and `checkInfection()` update the position and infection status of the individual.  `draw()` then draws the dot to the canvas.

```js
class Dot {
    constructor(index) {
        this.dead = false;
        this.inoculated = index !== 0 && random() < inoculation_rate;
        this.immune = this.inoculated && random() < inoculation_efficacy;
        this.infected = index === 0;
        this.recovered = false;
        this.color = this.getColor();
        this.position = createVector(random(width), random(height));
        this.infectionTimer = this.infected ? recovery_time : 0;
    }

    getColor() {
        if (this.dead) return 'gray';
        if (this.recovered) return 'green';
        if (this.infected) return 'red';
        return this.inoculated ? 'blue' : 'black';
    }

    updatePosition() {
        if (!this.dead) {
            let movement = createVector(random(-1, 1), random(-1, 1));
            this.position.add(movement);
            this.position.x = constrain(this.position.x, 0, width);
            this.position.y = constrain(this.position.y, 0, height);
        }
    }

    checkInfection(dots) {
        if (this.infected && this.infectionTimer > 0) {
            this.infectionTimer--;
            if (this.infectionTimer === 0) {
                if (random() < mortality_rate) {
                    this.dead = true;
                } else {
                    this.infected = false;
                    this.recovered = true;
                    this.immune = true;
                }
                this.color = this.getColor();
            }
        } else if (!this.infected && !this.immune && !this.dead && !this.recovered) {
            dots.forEach(dot => {
                if (dot.infected && !dot.dead && this.position.dist(dot.position) < infection_distance) {
                    if (random() < infection_chance) {
                        this.infected = true;
                        this.infectionTimer = recovery_time;
                        this.color = 'red';
                        return;
                    }
                }
            });
        }
    }

    draw() {
        fill(this.color);
        noStroke();
        ellipse(this.position.x, this.position.y, 10, 10);
    }
}
```

The `setup()` and `draw()` functions then set up the simulation and progress through each time step, updating each dot's position and infection status.

```js
function setup() {
    canvas = createCanvas(400, 400);
    background(220);
    dots = [];
    for (let i = 0; i < num_dots; i++) {
        dots.push(new Dot(i));
    }
    frameRate(frame_rate);
}

function draw() {
    background(220);
    dots.forEach(dot => {
        dot.updatePosition();
        dot.checkInfection(dots);
        dot.draw();
    });
    updateCounters();
}
```

While a very simple model, the simulation shows how variables like inoculation rate and transmissibility can affect how much, and how quickly, a disease can spread.