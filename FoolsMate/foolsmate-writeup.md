# TryHackMe — Fool's Mate: Can You Bypass the Engine?

## Table of Contents
1. [Overview](#overview)
2. [Goal of the Room](#goal-of-the-room)
3. [Tools Used](#tools-used)
4. [Enumeration](#enumeration)
5. [Discovering the API Endpoint](#discovering-the-api-endpoint)
6. [Analyzing the Response](#analyzing-the-response)
7. [Finding the Winning Move](#finding-the-winning-move)
8. [Exploitation](#exploitation)
9. [Vulnerability Explanation](#vulnerability-explanation)
10. [Skills Demonstrated](#skills-demonstrated)
11. [Mitigation](#mitigation)
12. [Lessons Learned](#lessons-learned)

## Overview

The Fool's Mate room presents a chess web application where the goal is to achieve checkmate. Rather than winning the game through the frontend chessboard interface, the challenge involves identifying the difference between how the frontend chessboard and the backend validation handle moves.

- **Difficulty:** Level 1
- **Platform:** TryHackMe
- **Category:** Web Application / API Security / Business Logic

## Goal of the Room

The objective of this room was to:
- Analyze how the chess application communicates with its backend
- Identify the API responsible for processing moves
- Determine whether the backend independently validates moves or trusts the client
- Achieve checkmate by any means necessary, including bypassing the intended game flow

## Tools Used

- Firefox Developer Tools
- Network Inspector
- Browser Request Editor

## Enumeration

The application was accessed at `http://10.67.167.143`. Firefox Developer Tools (F12) were opened, and the Network tab was used to monitor requests made by the page while interacting with the chessboard.

## Discovering the API Endpoint

<img width="900" height="501" alt="image" src="https://github.com/user-attachments/assets/ca59c236-52e4-4078-8756-4e8a6c81d422" />


While making a move on the board, a POST request was identified:

```
POST /api/move
```

The request body contained the chess coordinates for the move:

```json
{"from":"f2","to":"f3"}
```

This confirmed that every move made on the frontend board was being sent to a backend API endpoint for processing.

## Analyzing the Response

Submitting an incorrect/test move returned a FEN (Forsyth–Edwards Notation) string revealing the current board position:

```
6k1/5ppp/8/8/8/5PP1/7P/R5K1 w - - 1 3
```

Since the API returned the full board state, it was possible to read and interpret the exact position of every piece directly from the response.

## Finding the Winning Move

<img width="900" height="258" alt="image" src="https://github.com/user-attachments/assets/5ab6bb89-6075-4198-a5ee-30376a3e5110" />


The FEN showed it was White's turn to move. Based on the board position, the correct checkmate move was identified: moving the rook from `a1` to `a8`.

```
Ra8#
```

## Exploitation

The API request was crafted and sent directly, bypassing the frontend chessboard entirely:

```json
{"from":"a1","to":"a8"}
```

The server accepted the move and returned a response indicating **checkmate**, with **white** as the winner — confirming that the backend trusted whatever move coordinates were sent to it, regardless of whether the move followed a legitimate game sequence on the frontend.

## Vulnerability Explanation

The challenge demonstrated a **business logic flaw**. The backend API exposed the full chess board state and accepted move requests directly, rather than relying solely on the frontend interface to enforce game rules and turn order. Because the server did not independently verify that the submitted move was a legitimate continuation of the game, the checkmate condition could be triggered by sending a single crafted API request.

## Skills Demonstrated

- Web application traffic analysis using browser developer tools
- API endpoint discovery
- Reading and interpreting FEN chess notation
- Crafting and sending manual API requests
- Identifying business logic vulnerabilities

## Mitigation

- Enforce move validation and game-state logic entirely on the backend, independent of frontend input
- Track legitimate turn order and reject out-of-sequence or unauthorized moves
- Avoid exposing full internal game state in API responses where not strictly necessary
- Apply server-side sanity checks that verify a submitted move is a legal continuation of the current game

## Lessons Learned

- Inspect API requests during web application testing
- Client-side interfaces should not be trusted
- API responses may reveal useful information
- Business logic vulnerabilities can bypass normal application workflows
