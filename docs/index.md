# TMDB API Guide

A practical guide to The Movie Database API, written for developers making their first requests.

TMDB provides a free REST API for movie and TV data — titles, cast, images, ratings, and search. This guide covers authentication, the requests you'll make most often, and what goes wrong when they fail.

## Where to start

If you have never called the API before, begin with the [Quickstart](quickstart.md). It takes about five minutes and ends with a working search request.

If you already have a key and want specifics, the reference section documents endpoints, error codes, and rate limits.

## What this guide assumes

You can run commands in a terminal, and you have a free TMDB account. No prior API experience is needed. Examples are shown in curl, Python, and JavaScript.

## What this guide is not

This is not a replacement for TMDB's official reference. It is a working guide — the things that trip people up, the responses you actually get back, and how to read them.