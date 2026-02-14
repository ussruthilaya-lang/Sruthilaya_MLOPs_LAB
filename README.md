# Lab 1 – API Labs Submission 

- Course: MLOps
- Lab: API Labs (FastAPI + Streamlit)
- Submission Date: 30 Jan 2026

## Overview

This lab focuses on building and deploying an API-based machine learning system.

I trained a flower species classification model (Iris with extended Lily classes), exposed it through a FastAPI backend, and built a Streamlit dashboard to interact with the API. The backend was containerized and deployed using Google Cloud Build and Cloud Run.

The goal of this lab was to understand end-to-end API-based model serving and real cloud deployment constraints.

## What was implemented

- Trained a classification model on Iris + extended Lily dataset

- Built a FastAPI backend for model inference

- Exposed a /predict endpoint returning species, family, confidence, and probabilities

- Dockerized the FastAPI service

- Deployed the backend to Google Cloud Run using Cloud Build

- Built a Streamlit dashboard as a thin client for the API

# Lab 2 – Airflow ML Pipeline

- Course: MLOps
- Lab: Airflow Labs (FastAPI + Streamlit)
- Submission Date: 14 Feb 2026

## Overview

In this lab, I worked on building and modifying an Airflow DAG that runs a machine learning pipeline using clustering on customer credit card data.

## What was implemented

The pipeline basically:

Loads dataset

Preprocesses data

Trains clustering model

Finds optimal number of clusters

Saves model

Loads model and predicts

But during the lab, I realised some architectural issues in the original workflow and I improved them.

This lab helped me understand how ML pipelines actually work inside orchestration tools like Airflow.
