# NGV/survey an application based on NGV to manage 3d assets

## Why NGV?

At C2C we have more than 10 years experience with building 3D web applications.
Originally, around geoportals and fully custom solutions.

Since then technology has matured and the market grown. We felt we were not capitalizing on our expertise and code, so we started NGV, our opensource framework to create 3D applications.
This framework allows to leverage reusable code to create applications tailored to a particular use case.

## NGV/Survey

Today I present NGV/Survey: an application based on NGV to manage 3d assets.

This solution was presented at the FOSS4G in July.
https://talks.osgeo.org/foss4g-europe-2025/talk/review/CKHZL77YWEP3ZWYQ79BTBLWU3KDF3UXR
[see]

Here is how it looks:
https://ngv.labs.camptocamp.com/src/apps/survey/index.html
[see]

## Features

HES is a Scotish public body that takes care of "heritages", notably castles / churches / ....
At this stage, the solution allows to:
- display the 3d buildings (outside, but also inside);
- navigate the buildings in 3D;
- display / add / edit annotations put anywhere in 3D (a wall, roof, bench, ...)
- do all that while offline and synchronize the changes to their backend server.

## Architecture

What is interesting with our solution is that:
- [1] we reuse a tons of code from the NGV framework;
- [2] on top of that we have the survey logics;
- [3] and finally the HES specificities, which is separated and fit in a single file.

[1] https://github.com/geoblocks/ngv
[2] https://github.com/geoblocks/ngv/blob/master/src/apps/survey/index.ts
[3] https://github.com/geoblocks/ngv/blob/master/src/apps/survey/demoSurveyConfig.ts

## Use it!

So, we have created a 3d asset management solution that can easily be used by others.

Think about NGV