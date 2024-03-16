# Harvey Data Room MVP

## Development

Please refer to the `Makefile` in each frontend and backend folder to understand how you can get started with this project.

## Overview

The MVP supports all of the create, delete, rename, move operations outlined in the requirements, and keeps data compartmentalized based on user and data room. For demonstration purposes, authentication is based on a single username field that is used to keep data segregated.

## Collaboration

Users will want to include specific stakeholders in specific data rooms to facilitate specific work.

- Data rooms is a fundamental component of the product, allowing each user to create and manage multiple data rooms.
- The data model is designed such that switching to a many-many relationship between users and data rooms in the future is less costly.
- Likewise, separating read and write permissions at the data room level yields a much simpler UX and is lower short term lift than a file/folder-level permissioning scheme.

Resource Relationships:

    User
    |
    |-- DataRoom 1
    |   |
    |   |-- File 1
    |   |
    |   |-- Folder 1
    |       |
    |       |-- File 2
    |
    |-- DataRoom 2
        |
        |-- File 4
        |
        |-- Folder 3
            |
            |-- File 5
            |
            |-- File 6
            |
            |-- Folder 4
                |
                |-- File 7

## Performance and Scalability

There are many users, not as many datarooms and files/folders per user (maybe thousands).

- Data is highly relational, kept in User, DataRoom, Folder, and File SQL tables (database/models.py).
- Uploaded PDFs are stored in mounted filesystem (encrypted and compartmentalized per user and data room) in the backend. Pointers to data in storage are stored in the File table.
  - This can easily be swapped out for a cloud blob store + fully-qualified URLs stored in SQL, settled on this solution in the interest of time.
- Files and folders hold pointers to their parent folders (sometimes null) and their parent data room. We optimize queries for finding children files and folders quickly by indexing parent IDs, which facilitates edits/moves and simplifies navigation to lazy fetching and following of next state.

<selectedDataRoom, selectedFolder, selectedFile> (UI state)

- We index on username and email for quick validation of uniqueness during signup, and on owner ID in every table in order to facilitate future ownerhip/permissioned lookups within data rooms that may have many contributors in the future. Today, this helps us avoid sweeping the DataRooms table on initial load of a user's data rooms.
- Orphaned folders and files (in the event of a folder or data room deletion) are looked up in constant time and automatically deleted.

  File System the User Sees:

  |-- root
  |-- user_1
  |-- data_room_1
  |-- folder_1
  |-- folder_2
  |-- folder_3
  |-- folder_4
  |-- file_1

  Location of File on Disk:

  |-- root
  |-- user_1
  |-- data_room_1
  |-- folder_4 (user, data room, parent folder, file)
  |-- file_1

## Security

- Individuals are not always on identical need-to-know bases with regard to sensitive data.
- File/folder ownership is verified on action. You can see the beginnings of a permissions model in the backend (but not yet used in the frontend) in anticipation of separate view and edit permissions.
- Data is encrypted at rest.
- All actions are transactional.
- "Auth"/session management (JWT).

## Exploration and Future Work (Most Dire -> Most Exploratory)

- Styling
- Sanitized input fields
- Pagination
- Drag and drop display items
- OAUTH
- File-level sign off (request and confirmation)
- File preview/thumbnails
- Chunked/bulk upload (+ progress bar)
- Disable clipboard/print client-side
- End to end encryption with SSO/Yubikey

## Stack

- React/TS Frontend
- Flask/Python Backend (GUnicorn)
- SQLite3 DataBase (Flask-SQLAlchemy)
- File Storage on File System
