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
        text lyrics
        string title
        string albumTitle
        string imageUrl "nullable"
        date releasedDate "@IsDateString()"
        string duration "int as for miliseconds" 
        string youtubeUrl "for video"
        string audioUrl "audio for scoring"
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
        string spotifyId "id from spotify api"
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
    %% Clients
    subgraph Clients
        MobileApp["React Native (Mobile App)"]
    end

    %% Backend Services
    subgraph NestJS Backend
        CoreLogic["Core Logic (Auth, Playlist, Song, etc.)"]
        AILogic["AI Scoring Logic (30s Comparison, Result Aggregation)"]
    end

    %% OAuth Google
        subgraph Auth
        GoogleOAuth["Google OAuth 2.0"]
    end


    %% External Data APIs
    subgraph Data APIs
        YouTube["YouTube API"]
        Spotify["Spotify API"]
        Deezer["Deezer API"]
        LyricsAPI["Lyrics.ovh"]
    end

    %% AI Microservices (Self-hosted)
    subgraph Self-hosted AI Microservices
        Whisper["Whisper (Transcription)"]
        BasicPitch["Basic Pitch (Pitch-to-MIDI)"]
    end

    %% Cloud Services
    subgraph Firebase
        FCM["FCM (Notifications)"]
        Storage["Cloud Storage (Audio Uploads)"]
    end

    %% Database
    subgraph Database
        PostgreSQL["PostgreSQL (via TypeORM)"]
    end

    %% Flow
    MobileApp --> CoreLogic
    CoreLogic --> AILogic

    CoreLogic --> Firebase
    CoreLogic --> PostgreSQL

    AILogic --> Spotify
    AILogic --> Deezer
    AILogic --> YouTube
    AILogic --> LyricsAPI
    AILogic --> Whisper
    AILogic --> BasicPitch

    MobileApp --> FCM
    MobileApp --> Storage
    MobileApp --> GoogleOAuth --> CoreLogic

```
# EXTERNAL APIS

## SPOTIFY 

### Spotify API Client Credentals Flow

![Spotify API Flow](https://developer-assets.spotifycdn.com/images/documentation/web-api/auth-client-credentials.png)

### Spotify Sample JSON Response
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
## YOUTUBE

### Youtube Sample JSON Response
``` js
URL: https://www.googleapis.com/youtube/v3/videos?id=7lCDEYXw3mM&key=YOUR_API_KEY
     &part=snippet,statistics&fields=items(id,snippet,statistics)

Description: This example adds the fields parameter to remove all
             kind and etag properties from the API response.

API response:

{ 
 "videos": [
  {
   "id": "7lCDEYXw3mM",
   "snippet": { 
    "publishedAt": "2012-06-20T22:45:24.000Z",
    "channelId": "UC_x5XG1OV2P6uZZ5FSM9Ttw",
    "title": "Google I/O 101: Q&A On Using Google APIs",
    "description": "Antonio Fuentes speaks to us and takes questions on working with Google APIs and OAuth 2.0.",
    "thumbnails": {
     "default": {
      "url": "https://i.ytimg.com/vi/7lCDEYXw3mM/default.jpg"
     },
     "medium": {
      "url": "https://i.ytimg.com/vi/7lCDEYXw3mM/mqdefault.jpg"
     },
     "high": {
      "url": "https://i.ytimg.com/vi/7lCDEYXw3mM/hqdefault.jpg"
     }
    },
    "categoryId": "28"
   },
   "statistics": {
    "viewCount": "3057",
    "likeCount": "25",
    "dislikeCount": "0",
    "favoriteCount": "17",
    "commentCount": "12"
   }
  }
 ]
}
```

## GOOGLE OAUTH 2.0 FLOW

![Google OAuth 2.0 Flow](https://developers.google.com/static/identity/protocols/oauth2/images/flows/authorization-code.png)
