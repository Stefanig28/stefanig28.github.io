---
layout: post
title: rules of the game Galactic Syndicate
date: 2025-10-30 07:01:00
description: in this section, I designed the rules for my game Galactic Syndicate.
tags: formatting images game
categories: posts
thumbnail: assets/img/galactic_sydicate_demo.jpg
pretty_table: true
---

## Galactic Syndicate

### Rise in the Smuggling Nebula

In a dystopian and technologically advanced future, control of the galaxy no longer lies in the hands of governments, but in vast and shadowy networks of illegal trade and smuggling of rare resources. Power is measured in influence and credits, and only the most ruthless leaders rise to the top of the Galactic Syndicate (an alliance of the universe’s most powerful criminal empires).

In this world, survival depends on eliminating rivals, expanding control, and silencing anyone who dares to challenge your reign. The last cartel standing will decide the fate of the galaxy.

## Core Game Rules

The game is played in a series of turns, moving clockwise, until only one player remains with Influence.

### Game Objective and Elimination

Objective: Be the last player remaining with at least one Influence Card (face-down card) remaining in your possession.

Influence: You start the game with two face-down Influence Cards. Losing Influence means revealing a card and placing it face-up, where its power is lost.

Elimination: A player is immediately eliminated from the game when they lose their second Influence (both cards are face-up). All their remaining Credits are returned to the Bank.

### Structure of a Turn

On your turn, you must choose and announce one of the seven possible actions.

### General Actions (Always Available)

These actions do not require an Influence Card and cannot be Challenged.

Income: Take 1 Credit from the Bank.

Foreign Aid: Take 2 Credits from the Bank. (This action can be Blocked by The Collector).

Coup (Cost 7 GC): Pay 7 Credits to the Bank. Force one target player to immediately lose 1 Influence. Cannot be Blocked.

Mandatory Rule: If you start your turn with 10 or more Credits, you must perform a Coup as your only action.

### Role Actions (Claiming Influence)

The player announces they are taking an action by declaring the corresponding Role (even if they don't possess the card this is the bluffing mechanic). The claim to this Role can be Challenged.

## Reaction Phase: Challenge and Block

After any player announces a Role Action or a Block, the other players can react.

### The Challenge (Doubt)

Any player can immediately Challenge the acting or blocking player if they suspect a lie. If the Challenged Player is telling the truth (HAS the card): The Challenged Player reveals the card, then shuffles it back into the Deck and draws a replacement card. The Challenging Player loses 1 Influence. The original Action/Block is successful. If the Challenged Player is bluffing (DOES NOT have the card): The Challenged Player loses 1 Influence. The original Action/Block fails and is canceled.

## Blocking (Counter-Action)

If an action is blockable, any other player can claim the appropriate Counter-Action Role. The acting player may then choose to Challenge the player attempting the Block. If the Block is successful (not challenged, or the Challenger loses), the original action is canceled.

## Characters and abilities

### 1. The Collector

The Collector is the Syndicate's ruthless financial enforcer. They are not interested in direct smuggling or elimination; their power lies in controlling the flow of Galactic Credits (CR) and ensuring all cartels pay their "taxes." Their presence guarantees that money continues to flow into the central coffers.

- Action: Taxes. The Collector takes 3 Credits directly from the Bank (Galactic Reserve).

- Counter-Action: Block Foreign Aid. Can stop any player's attempt to take the 2 Credits offered by the Foreign Aid action.

<div class="row mt-5 mb-5">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/the_collector.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

### 2. The Cyber-Hitman 

This specialist is the Syndicate's ultimate elimination tool. Equipped with military-grade cybernetic implants and advanced stealth training, The Cyber-Hitman is contracted to resolve influence problems that cannot be solved with money or negotiation. Their contract always requires a 3 Credit payment to activate their strike.

- Action: Assassinate (Cost 3 GC). Pay 3 Credits to the Bank to force one target player to immediately lose 1 Influence (reveal one of their cards).

- Counter-Action: None. (This action can only be blocked by The Commander).

<div class="row mt-5 mb-5">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/the_cyber_hitman.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

### 3. The Smuggler 

The Smuggler is a daring pilot and expert in illegal trade in the Outer Rim. They thrive on intercepting cargo and diverting trade routes. Their skill allows them to take valuable resources from other cartel bosses right out of their hands.

- Action: Steal. The Smuggler takes 2 Credits directly from any other player.

- Counter-Action: Block Stealing. Can block any player's attempt to Steal Credits.

<div class="row mt-5 mb-5">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/the_smuggler.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

### 4. The Coalition Spy 

This character is a double agent or an infiltrator from a powerful, neutral faction. Their true power lies in data manipulation and the constant realignment of loyalties. The Spy can not only steal resources but also constantly improves their position and contacts by exchanging Influence cards in the Deck.

- Action: Exchange. The Spy draws 2 new Influence Cards from the Deck, looks at them (along with their current hand), and returns any 2 cards to the Deck (which is then shuffled).

- Counter-Action: Block Stealing. Can block any player's attempt to Steal Credits.

<div class="row mt-5 mb-5">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/the_coalition_spy.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

### 5. The Commander 

The Commander represents the highest authority and the last line of protection within the Syndicate. Although they do not have an offensive action of their own on their turn, their influence is so great that they can stop any direct elimination threat against their allies.

- Action: None. (The Commander does not have an active turn action).

- Counter-Action: Block Assassination. Can block The Cyber-Hitman's Assassinate action, protecting the targeted player.

<div class="row mt-5 mb-5">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/the_commander.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

| Character | Main Action (Ability) | Counter Action (Block) |
| :----------- | :------------: | ------------: |
| The Collector | Taxes: Take 3 Galactic Credits from the Bank. | Block Foreign Aid |
| The Cyber-Hitman | Assassinate (Cost 3 GC): Force one player to lose 1 Influence (Card). |  |
| The Smuggler | Steal: Take 2 Galactic Credits from another player. | Block Stealing (Can be blocked by another Smuggler or the Coalition Spy) |
| The Coalition Spy | Exchange: Draw 2 new Influence Cards from the Deck, and return 2 cards (your choice) to the Deck. | Block Stealing (Can be blocked by The Smuggler or the Coalition Spy) |
| The Commander |  |  Block Assassination (Blocks The Cyber-Hitman's action) |

### Estructura

I want the game to start with a card-dealing effect. Each player will have two cards that appear slightly raised, as if they were holding them face up in their hand. When a player loses an influence card that is turned face up on the table, the other players' cards will be turned face up, and each player will always be able to see their own cards. A list of actions should appear on the player's screen, and they can choose any action. However, the actions associated with the cards they were dealt should be more visible or at the top of the list so the player knows that choosing those cards indicates they are lying. There should also be a "doubt" button that is always active. This is just one idea of ​​what it could look like.

<div class="row mt-5 mb-5">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/galactic_sydicate_demo.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>