# DATABASE SCHEMA
``` mermaid
erDiagram
    users {
        uuid id PK "auto inc"
        string displayName
        string email
        string password "hashed"
        string imageUrl "nullable"
        timestamp createdAt "DEFAULT CURRENT_TIMESTAMP"
        string adminSecret "nullable"
    }
    friends {
        uuid userId_1 FK "part of PK"
        uuid userId_2 FK "part of PK"
        enum status "'pending' | 'accepted' | 'rejected'"
        timestamp createdAt "DEFAULT CURRENT_TIMESTAMP"
    }
    songs {
        uuid id PK "auto inc"
        string title
        string albumTitle
        string imageUrl "nullable"
        date releasedDate "@IsDateString()"
        string duration "int as for miliseconds" 
        string youtubeUrl "for video"
        string spotifyApiUrl "audio for scoring"
    }
    playlists {
        uuid id PK "auto inc"
        string title "playlist's title"
        string imageUrl "nullable"
        text description
        uuid userId FK
    }
    playlist_songs {
        uuid playlistId FK "part of PK"
        uuid songId FK "part of PK"
    }
    artists {
        uuid id PK "auto inc"
        string name
        string imageUrl "nullable"
        int popularity "out of 100"
    }
    artist_songs {
        uuid artistId FK "part of PK"
        uuid songId FK "part of PK"
    }
    scores {
        uuid id PK "auto inc"
        uuid userId FK
        uuid songId FK
        string audioUrl "user's recording"
        float finalScore
        timestamp createdAt "DEFAULT CURRENT_TIMESTAMP"
    }
    notifications {
        uuid id PK "auto inc"
        uuid userId FK
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
    end

    %% Database
    subgraph Database
        PostgreSQL["PostgreSQL (TypeORM)"]
    end

    %% Flow
    
    MobileApp --> CoreLogic --> AILogic


    AILogic --> YouTube
    AILogic --> Spotify
    AILogic --> PostgreSQL

```
# SPOTIFY API CLIENT CREDENTIALS FLOW

![Spotify API Flow](https://developer-assets.spotifycdn.com/images/documentation/web-api/auth-client-credentials.png)

## Spotify Sample JSON Response
``` js
{
  "external_urls": {
    "spotify": "https://open.spotify.com/artist/4Z8W4fKeB5YxbusRsdQVPb"
  },
  "followers": {
    "href": null,
    "total": 7625607
  },
  "genres": [
    "alternative rock",
    "art rock",
    "melancholia",
    "oxford indie",
    "permanent wave",
    "rock"
  ],
  "href": "https://api.spotify.com/v1/artists/4Z8W4fKeB5YxbusRsdQVPb",
  "id": "4Z8W4fKeB5YxbusRsdQVPb",
  "images": [
    {
      "height": 640,
      "url": "https://i.scdn.co/image/ab6761610000e5eba03696716c9ee605006047fd",
      "width": 640
    },
    {
      "height": 320,
      "url": "https://i.scdn.co/image/ab67616100005174a03696716c9ee605006047fd",
      "width": 320
    },
    {
      "height": 160,
      "url": "https://i.scdn.co/image/ab6761610000f178a03696716c9ee605006047fd",
      "width": 160
    }
  ],
  "name": "Radiohead",
  "popularity": 79,
  "type": "artist",
  "uri": "spotify:artist:4Z8W4fKeB5YxbusRsdQVPb"
}
```
