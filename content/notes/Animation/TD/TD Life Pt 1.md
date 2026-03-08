---
tags:
  - feature-animation
  - technical-director
---

## Support and Simple Scripts

I started my career as a Technical Director, which at DWA is essentially a support focused engineer who gets embedding within an artist department to make sure movies go smoothly. In the surfacing department, my tasks were focused on fixing technical errors, developing scripts to automate non-artistic tasks, and helping to improve the health of the pipeline as a whole. There was a lot that was cool about being a TD that has proven useful later on in my career:

- It requires a lot of on the job learning
- You have to master context-switching ASAP
- You're in constant communication with your users
- You have to learn to be efficient when coding

In those days, surfacing was not only responsible for look dev (textures, materials, and some micro-geometry) but also acted as the chokepoint for front-end assets on the movie. Every character and environment asset would pass through surfacing, and would be validated & rendered. This meant that when we first adopted Torch (a short lived in-house developed lighting package, upgrading from Light, a glorified spreadsheet), surfacing was responsible for asset conversion to the new format. In addition, surfacing was often responsible for helping to maintain the crowds character generation pipeline (known as the catalog), which meant it could be a pretty hectic department.

### Wrapping basic Linux functionality

I quickly learned the value of leveraging Linux command line tools to achieve quick and dirty functionality; one of my first tasks on The Croods was to give artists a GUI to schedule a specific type of render to occur later in the evening after all the artists on the production would most likely have checked in their work. There was a bit of nuance to this beyond just setting up a cron job, but providing a gui around crons was a surprisingly powerful concept for artists.

### Getting addicted to templates

I eventually learned to embrace scripts that could reference and leverage templates. At the time, the primary asset format at DWA was ADB (assets) + ASG (scenes), which was a plain text format. ADB would hold definitions of assets, including collections of assets, as well as shading networks; in a lot of ways it was like USD (Universal Scene Description) before USD. When I started, eye and hair setup was done by running scripts which did fairly hard-coded setups on a character asset, which had to be manually tweaked when standards changed or if a character had an atypical number of eyes.

I realized that both of these, as well as a few others, were a similar sort of problem, and one that was obscured from the artist. I unified the code to be an ADB template engine, and made the scripts exposed to the user much simpler and more flexible. Now, artists could easily modify and update templates and not have to worry about how many eyes a character had. 

This kicked off a career long love of exposing functionality as templates that I haven't been able to quit. It also led to my contributions to an iteration of the Global Material Library at DreamWorks, which got me and a few other colleagues recognition in the form of a Technical Achievement Award, which is a very pointy piece of glass.

