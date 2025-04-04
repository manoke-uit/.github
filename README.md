# DATABASE SCHEMA
``` mermaid
erDiagram
    users {
        int id PK "auto gen"
        string displayName
        string email
        string password "hashed"
        string imageUrl "nullable"
        timestamp createdAt "DEFAULT CURRENT_TIMESTAMP"
    }
    friends {
        int userId_1 FK "part of PK"
        int userId_2 FK "part of PK"
        enum status "'pending' | 'accepted' | 'rejected'"
    }
    songs {
        int id PK "auto gen"
        string title
        string album
        string imageUrl "nullable"
        string releasedDate "@IsDateString()"
        string duration "@IsMilitaryTime()" 
        string youtubeUrl "for video"
        string spotifyApiUrl "audio for scoring"
    }
    playlists {
        int id PK "auto gen"
        string title "playlist's title"
        string imageUrl "nullable"
        text description
        string coverUrl "playlist's photo"
        int userId
    }
    playlist_songs {
        int playlistId FK "part of PK"
        int songId FK "part of PK"
    }
    artists {
        int id PK "auto gen"
        string name
        string imageUrl "nullable"
        int popularity "out of 100"
    }
    artist_songs {
        int artistId FK "part of PK"
        int songId FK "part of PK"
    }
    scores {
        uuid id PK
        int userId FK
        int songId FK
        string audio_url "user's recording"
        float finalScore
        timestamp createdAt "DEFAULT CURRENT_TIMESTAMP"
    }
    notifications {
        uuid id PK
        int userId FK
        string title
        text description
        timestamp createdAt "DEFAULT CURRENT_TIMESTAMP"
        boolean isRead
    }
    
    songs }o--|| artist_songs: belongs
    users }o..|| notifications: has
    users }o--|| friends: user_id_1
    users }o--|| friends: user_id_2
    users }|..|| playlists: has
    songs }o..|| scores: has
    users }o..|| scores: has
    playlists }o--|| playlist_songs: has
    songs }o--|| playlist_songs: has
    artists }o--|| artist_songs: owns
    
```
# ARCHITECTURE

``` mermaid
flowchart TD
    %% Consumers
    subgraph Clients
        MobileApp["React Native"]
    end

    %% Backend Services Grouped
    subgraph NestJS Server
        CoreLogic["Core Business Services (Auth, Playlist, Song, etc.)"]
        AILogic["AI Services (Custom Scoring Logic, Pitch Analysis, etc.)"]
    end

    %% External APIs
    subgraph External APIs
        YouTube["YouTube API"]
        Spotify["Spotify API"]
        Genius["Genius API"]
    end

    %% Database
    subgraph Database
        PostgreSQL["PostgreSQL (TypeORM)"]
    end

    %% Flow
    MobileApp --> CoreLogic
    MobileApp --> AILogic

    CoreLogic --> YouTube
    CoreLogic --> Spotify
    CoreLogic --> Genius

    CoreLogic --> PostgreSQL
    AILogic --> PostgreSQL

```
