---
description: Welcome to the SYMMIO smart contracts documentation.
---

# Overview

## Introduction

Symmetrical Contracts are a new financial trading primitive for trading derivatives permissionlessly and in a trustless manner on the blockchain, inspired by TradFi "Bilateral Agreements". SYMMIO uses **intent-centric** architecture combined with bilateral OTC to create truly permissionless digitized derivatives for the blockchain economy.

Intent-centric means, there is a `PartyA` that requests a trade and a `PartyB` that responds, a PartyA & B match results in both sides putting up collateral (hence bilateral) to execute the trade. The core contracts focus on the sending, receiving, locking and accepting of quotes, as well as the management of the provided collateral by both parties, additionally it also regulates how 3rd Party Liquidators can use Oracles to flag parties as liquidatable in case they fail to provide sufficient capital.

The core contracts do not provide a matching engine-- the matching of PartyA and PartyBs can be done via decentralized frontends or decentralized driver architecture, as external services. Matching can easily be facilitated by providing an array of whitelisted PartyBs in the quote.

This document is a technical guide for SYMM core contracts v.0.8.2, explaining the system's architecture, functionalities, and how to interact with it. It covers the intent-centric design, collateral management, and roles of oracles and liquidators, offering a comprehensive understanding of SYMMIO's permissionless derivatives on the blockchain.

## Setup Instructions

To setup the testing environment:

* Clone the [Github Repo](https://github.com/SYMM-IO/protocol-core).
* Copy the `.env.example` file, name it `.env`, and then provide the required variables in that .env file.
* Then you can run static tests by using `npx hardhat test`
* Also, there is a hardhat task for deploying the contracts. You can run it with `npx hardhat deploy:diamond`
