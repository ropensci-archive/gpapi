[![Project Status: WIP – Initial development is in progress, but there has not yet been a stable, usable release suitable for the public.](http://www.repostatus.org/badges/latest/wip.svg)](http://www.repostatus.org/#wip)

gpapi
=====

**goodpractice as a service**

Base URL: TBD

API routes

* `/` (GET) - metadata on API, list routes
* `/gp` (POST) - run `goodpractice` on a pkg, auth required
* `/status/<job id>` (GET) - get job status
* `/jobs` (GET) - list jobs running, auth required
* `/job/<job id>` (GET, POST, PUT, DELETE) - get a job, auth required

## how it all fits together

> using docker image rocker/ropensci

- request comes in to `/gp` (`api` docker container)
    - put new job into scheduler (to `scheduler` docker container)
    - API response (in `application/json`) with metadata and info to fetch results later
- clone package
- cd into package root
- inside docker container:
    - run gp:
        - `x <- goodpractice::gp()`
    - collect output into a JSON file: 
        - `goodpractice::export_json(x, "ropensci_check_pkgname_version_YY-MM-DD-HH.json")`
- kill container
- post JSON file to Amazon S3 bucket
    - s3 cli client?
- ping GitHub issue with S3 link to results
    -  ruby git client? or another?
- (ping github issue with summary of results?)

How will this whole pipeline be triggered? An issue opening?  probably not. Probably an editor triggering, but how?

## tech

- API: Ruby Sinatra (an `api` container)
- Scheduling/Jobs: Sidekiq (in its own container)
- Running jobs: Docker
    - via Ruby with:
        - `docker-api` gem
        - `git` gem <https://github.com/ruby-git/ruby-git>
        - `aws-sdk-s3` gem
        - `octokit` gem
