# Movie Recommender Application Architecture

This document provides a comprehensive overview of the system's design, data structures, and behavioral flows.

---

## 1. Data Flow Diagram (DFD) - Level 0
Represents how data moves between the user, the admin, and the local storage system.

```mermaid
flowchart TD
    User[User] -->|Search Query, Clicks| System((Movie Recommender System))
    System -->|Search Results, Similar Recommendations| User
    
    Admin[Admin] -->|Password, New Ratings| System
    System -->|Update Confirmation| Admin
    
    System <-->|Read/Write Movie Data| DB[(LocalStorage JSON DB)]

erDiagram
    MEDIA_ITEM {
        string name PK
        float rating
        string description
        string actors
        string genre
        string trailer
        string item_type "Movie or WebSeries"
    }
    
    MEDIA_ITEM ||--o{ MEDIA_ITEM : "has similarity score with"

flowchart LR
    %% Actors
    U([User])
    A([Admin])
    
    %% Use Cases
    Search(Search Movies/Series)
    View(View Movie Details & Recommendations)
    Watch(Watch Trailer)
    Login(Access Admin Panel)
    Edit(Edit Ratings)
    
    %% Relationships
    U --> Search
    U --> View
    U --> Watch
    
    A --> Login
    Login --> Edit
    
    %% Admin inherits User capabilities
    A -.-> U

classDiagram
    class ApplicationDB {
        +Array movies
        +Array webseries
        +initDB()
        +saveRating(name, rating)
    }
    
    class MediaItem {
        +String name
        +Float rating
        +String description
        +String actors
        +String genre
        +String trailer
    }
    
    class RecommenderEngine {
        +searchMovies(query)
        +calculateSimilarity(baseItem, compareItem)
        +showPopup(name)
        +displayResults(items)
    }

    ApplicationDB "1" *-- "*" MediaItem : contains
    RecommenderEngine --> ApplicationDB : queries


classDiagram
    class `db: ApplicationDB` {
        movies = [...]
        webseries = [...]
    }
    
    class `movie1: MediaItem` {
        name = "The Godfather"
        rating = 9.2
        genre = "Drama"
    }
    
    class `series1: MediaItem` {
        name = "Breaking Bad"
        rating = 9.5
        genre = "Drama"
    }
    
    `db: ApplicationDB` --> `movie1: MediaItem`
    `db: ApplicationDB` --> `series1: MediaItem`

sequenceDiagram
    actor User
    participant DOM as UI (DOM)
    participant JS as App Logic (JS)
    participant DB as LocalStorage
    
    User->>DOM: Types query & clicks Search
    DOM->>JS: handleSearch()
    JS->>DB: Fetch movies & series
    DB-->>JS: Return JSON arrays
    JS->>JS: filter matches
    JS->>DOM: displayResults(matches)
    DOM-->>User: Renders movie cards
    
    User->>DOM: Clicks on "The Dark Knight"
    DOM->>JS: showPopup('The Dark Knight')
    JS->>JS: calculateSimilarity() for all items
    JS->>JS: Sort & slice top 18 related
    JS->>DOM: Render popup overlay
    DOM-->>User: Displays details & related grid


stateDiagram-v2
    [*] --> EnterSearchQuery
    EnterSearchQuery --> CheckQueryLength
    
    CheckQueryLength --> LoadAllItems: Query is empty
    CheckQueryLength --> FilterItems: Query has text
    
    LoadAllItems --> DisplayCards
    FilterItems --> DisplayCards
    
    DisplayCards --> UserClicksItem
    UserClicksItem --> FindItemInDB
    FindItemInDB --> CalculateSimilarities
    CalculateSimilarities --> SortBySimilarityScore
    SortBySimilarityScore --> RenderEnlargedPopup
    RenderEnlargedPopup --> [*]



flowchart TD
    User((User))
    UI[User Interface / DOM]
    Logic[Recommender Logic]
    DB[(Data Store)]

    User -- "1: inputs search term" --> UI
    UI -- "2: triggers searchMovies()" --> Logic
    Logic -- "3: requests all items" --> DB
    DB -- "4: returns JSON" --> Logic
    Logic -- "5: returns formatted HTML strings" --> UI
    UI -- "6: displays rendered cards" --> User

