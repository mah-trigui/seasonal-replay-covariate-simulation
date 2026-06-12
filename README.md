# Seasonal Replay Covariate Simulation

A project page about approximating missing future covariates by replaying prior seasonal structure and rescaling it with recent year-over-year change.

## Project Website

[View Project Site](https://mah-trigui.github.io/seasonal-replay-covariate-simulation/)

## Overview

This project focuses on a practical deployment problem in the Nairobi crash prediction pipeline.

Some important covariates — especially Uber Movement traffic speeds and weather variables — were not available for the future prediction period.

Rather than dropping those features or freezing them at their last observed values, the pipeline used a structured approximation.

## Core Idea

The approximation strategy was:

1. take the same seasonal window from the previous year
2. compute the recent year-over-year trend ratio
3. replay last year’s seasonal pattern at the adjusted level

This preserved:
- seasonal shape
- relative segment structure
- current level shift

## Architecture

![Architecture](images/architecture.jpg)

## Selected Implementation Logic

```r
uber_2018_h2 <- uber_by[year == 2018 & month %in% 7:12]
uber_2019_h1 <- uber_by[year == 2019 & month %in% 1:6]
uber_2018_h1 <- uber_by[year == 2018 & month %in% 1:6]

trend_ratio <- mean(uber_2019_h1$speed, na.rm = TRUE) /
    mean(uber_2018_h1$speed, na.rm = TRUE)

uber_future <- copy(uber_2018_h2)
uber_future[, year := 2019L]
uber_future[, speed := speed * trend_ratio]
