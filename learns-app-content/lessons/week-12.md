## Introduction

### Topics Covered

- Pagination
- Routing
- React Router

### Lesson Objectives

By the end of this lesson, we will:

#### Objective 1: Pagination

- discuss the advantages and challenges related to paginating data to improve a user's experience
- describe how to divide large lists of data to present them to a user in digestible chunks
- formulate a technique to display paginated data in a component

#### Objective 2: Routing

- explain how a web browser's session history works while a user navigates a typical website
- describe how we can use the History API to interact with a browser's session history

#### Objective 3: react-router

- describe the purpose of react-router and how client side routing improves a user's experience with the app
- implement react-router to emulate page navigation and enable client side routing in a React application

## Discussion Topics

### Pagination

- pagination breaks lists of data up (array of objects of the same structure) into digestible chunks
  - Google search results
  - book pages (yeah??)
  - social media feeds
  - forum conversations
  - lists of products for sale
  - infinite scroll feeds - Pinterest, X, Tumblr, YouTube
    - these have some other mechanisms in place to optimize for memory and network requests but we're not going to focus on that - complex and different ways to approach it
- setup
  - create pagination component
    - simple forward, backward buttons, may add page count later
    - props currentPageData
  - listeners on buttons
  - handler functions wired to listeners
    - nextPage
    - backPage
  - useReducer
  - count of objects to display at a time - configuration
  - state variable holding:
    - value of current pagination depth - counter
    - current subset of displayed objects
    - initial state {items: [], currentPage: 0)}
  - reducer function that returns the subset of the full list
    - takes count, current pagination depth
    - calculates start + stop values
    - returns appropriate subset base on action received
      - increment +
        - disable for end of list
      - decrement -
        - disable for beginning of list
      - default
        - error handling escape hatch
  - add useReducer's dispatch to handler functions
    - create action objects in handler functions
    - dispatch action

### Routing

- SPAs
  - don't have page history but can behave similar to traditional website
    - pages doesn't reload like a traditional website when we navigate
    - fetch happens in the background + React re-renders interface
  - can have full screens which users navigate
    - think of all the aspects of a social media website
      - we'll look at Facebook since they developed and maintain React
        - news feed
        - marketplace
        - settings and privacy
    - tends to feel like a traditional website
      - would be better for users if they can continue to use familiar tools like back and forward buttons
- starting to work with data that spans "pages"
- browser session history
  - keeps track of pages visited in a browser window
  - allows us to navigate back and forward quickly
  - each tab or window opened has its own session history
- can emulate some of this with the browser's History API
  - properties:
    - **length**: number of elements in session history
    - **scrollRestoration**: auto | manual - allows web apps to explicitly set scroll restoration
    - **state**: null until pushState or replaceState are used - can contain anything that can be serialized
    - NOTE: for security reasons, the API does not give us access to urls
      - This would allow a site to see where all we have been previous to that site. Major privacy concern!
  - methods:
    - **back**: navigates back 1 page
    - **forward**: navigates forward 1 page
    - **go**: takes a number and navigates to a page in history relative to current one open
    - **pushState**: adds an entry to the session history stack
      - takes
        - **state object**
          - mandatory
          - anything serializable
        - **unused**
          - parameter is not used but still need it
          - programmers sometimes make mistakes that are hard to get out of software!!
          - pass it an empty string
        - **url**
          - optional
          - url for new history entry
            - relative urls resolved relative to the current url
            - absolute urls must be of same origin as current URL
          - if unspecified, automatically set to current URL
    - **replaceState**: replaces current history entry
      - takes state object, unused, and url - same as defined in pushState
- complex problem, especially to take full advantage of state object

### React Router

- 3rd party tool that emulates page navigation in SPAs
- makes use of the History API and history.state
- take some time to familiarize yourself with [React Router Terminology](https://reactrouter.com/en/main/start/concepts#definitions)
- a lot to learn about this library but we are going to focus on:
  - matching
    - defining routes using component trees
    - dynamic segments
    - ranking routes and making matches
  - rendering
    - outlets
    - index route
    - grouping routes with a layout route
  - navigating
    - user directed navigation
    - programmatic navigation
