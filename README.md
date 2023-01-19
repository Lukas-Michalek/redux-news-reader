# **Redux News Reader**

This project - a news reader in which users can view and comment on various articles, gave me an opportunity to practice using Redux Toolkit’s **`createAsyncThunk`** and **`createSlice`** utilities to add asynchronous functionality to Redux applications.

<br>

The main task of this project was to complete the comments feature. Whenever the featured article changes, the comments are being asynchronously fetch for that article and added to the store so they display under the article. Likewise, when a user submits a new comment, it will be submitted to the server via an asynchronous request and displayed in the article’s list of comments once it has been successfully created.




