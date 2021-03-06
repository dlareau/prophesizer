# Prophesizer

Prophesizer is part of the SIBR Archiving Pipeline™.

## Table of Contents
  * [What](#what)
    * [Non-Update Data](#non-update-data)
    * [Update Data](#update-data)
  * [Installation](#installation)


## What

Prophesizer runs continuously, waking every minute to ask Chronicler (https://github.com/xSke/Chronicler) for new data.
It does the following:

1) Get the latest data from the Blaseball `/games` endpoint and store it in the `games` table
2) Check the DB's `chronicler_meta` table to see what the last day/time processed was
3) Check Blaseball's `/simulationData` endpoint to see what the current season/day is
4) Batch-process any updates for days up to the current day
5) Incrementally ask Chronicler for updates from the current day

Typically on first run Prophesizer will batch-process many seasons of data into the DB, and only grab a minute's worth of updates every minute after that.

### Non-Update Data

Prophesizer also asks Chronicler for updates to `players` and `teams`:

1) Add entries to `teams` for any team data changes (like name changes)
2) Add entries to `team_roster` to reflect any lineup/rotation/etc changes
3) Add entries to `team_modifications` to reflect any new "tags" (like the blood bath tags) for a team
4) Add entries to `players` for any player data changes (names, attributes)
5) Add entries to `player_modifications` for any new "tags" (like SHELLED) for a player

### Update data

These are logs of the individual (one every ~4seconds) `streamData` updates for gameplay.

Prophesizer sends these through [Cauldron](https://github.com/Society-for-Internet-Blaseball-Research/Cauldron) to convert them into SIBR's "Game Event" format.

Game Events roughly correspond to one at-bat (with some exceptions like a runner caught stealing) and are more easily queried for statistics than the raw JSON updates from the stream.

These Game Events are added to the `game_events` table, with child tables `game_event_base_runners` for baserunning information and `outcomes` for Outcome (incineration, peanuts, partying, etc).

## Installation

The following instructions are written for Windows PCs.

1. Prophesizer depends on [git](https://git-scm.com/), [PostgreSQL](https://www.postgresql.org/), [Visual Studio Code](https://code.visualstudio.com/), and [Node.js](https://nodejs.org/en/). You can manually download and install all of these, but if you have the package manager [Chocolatey](https://chocolatey.org/) installed, you can automatically install these tools by opening a prompt (cmd/powershell) as administrator and running: `choco install git postgresql vscode nodejs`
2. Use git to clone Prophesizer from github into your desired directory: `git clone https://github.com/Society-for-Internet-Blaseball-Research/prophesizer/`. 
3. Set the environment PSQL_CONNECTION_STRING to "Host=localhost;username=[postgres username, default 'postgres'];password=[postgres password];database=blaseball", with `setx`, making appropriate changes if any are necessary.
4. Make sure psql is added to PATH (you can test by typing it in as a command), and your C# connection string environment variable is correctly set.
5. Using pgAdmin or `psql`, create a database named `blaseball`.
6. Run the command `create extension unaccent` on the `blaseball` database.
7. In the directory prophesizer\\db, run `node schema.js load` to load the schema Prophesizer expects into the database.
8. Compile and run Prophesizer from VS Code via File -> Open Folder, selecting Prophesizer's folder, going to 'Run' in the menu bar, and selecting "Run Without Debugging".
9. If at some point you make changes to the schema you wish to commit to a repository using `git`, in the directory prophesizer\\db, run `node schema.js dump` to dump your database's schema to schema.sql.
