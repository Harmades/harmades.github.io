+++
date = '2025-04-20T01:36:28-03:00'
title = 'Github Gameoff 2021'
cover = "/games/github-gameoff-2021.png"
+++

# Context: Github Game off

Game Off is an annual game jam celebrating open source created by Lee Reilly in 2012 and sponsored by GitHub. Participants are given the entire month of November to build a game based on a theme–individually or as a team. Inspired by the Global Game Jam, it encourages collaborative game development and promotes the use and sharing of open source software.

This year's theme was *BUG*.

# Game: [Grand Matrix Reloaded](https://fennelle.itch.io/grand-matrix-reloaded)

Welcome to Grand Matrix Reloaded!

In this game, you are Agent Smith, enrolled by the machines to suppress any upcoming threats.

8 Neo-programs have been found corrupting the Matrix. Destroy them before they destroy the World!

Objective: Eliminate all Neos before they cause too many bugs in the city.

The Machines have given you the available abilities:

Arrow Keys / WASD / ZQSD : Move.
Left Click: Shoot. The Machines generously gave you unlimited ammo.
Right Click: Enter / Leave vehicle. Use them to travel faster.
Enjoy full Freedom in the Matrix: drive, shoot, kill anyone on your path or focus on your objective: your choice!

The game features:

Fully destructible city aka the Matrix
4 vehicles to enjoy, driving at a different speed
Unlimited shooting action, no ammo needed
Fierce opponents, ready to kill you on sight!
Adrian, Bart and Emily Hope you will have fun playing Grand Matrix Reloaded, just as they had fun making this game!

# Play the game!

<iframe id="iframe" src="/github-gameoff-2021" width="960px" height="540px" frameborder="0" ></iframe>

<script>
    const iframe = document.getElementById('iframe');
    iframe.addEventListener('load', () => {
        iframe.contentWindow.document.addEventListener('keydown', function(event) {
            if (['ArrowUp', 'ArrowDown', 'ArrowLeft', 'ArrowRight'].includes(event.key)) {
                event.preventDefault();
            }
        });
    });
</script>

> If you're having trouble with the embedded version, try the js13k page [here](https://fennelle.itch.io/grand-matrix-reloaded)