
# **ContactForm.js**

```javascript
import React, { useState } from 'react';
import { useDispatch, useSelector } from 'react-redux';
import {
  createCommentIsPending,
  postCommentForArticleId
} from '../features/comments/commentsSlice';

export default function CommentForm({ articleId }) {
  const dispatch = useDispatch();
  const [comment, setComment] = useState('');
  
  // Declare isCreatePending here.
  const isCreatePending = useSelector(createCommentIsPending);

  const handleSubmit = (e) => {
    e.preventDefault();
    // dispatch your asynchronous action here!
    dispatch(postCommentForArticleId({articleId, comment}));
    setComment('');
  };

  return (
    <form onSubmit={handleSubmit}>
      <label for='comment' className='label'>
        Add Comment:
      </label>
      <div id='input-container'>
        <input
          id='comment'
          value={comment}
          onChange={(e) => setComment(e.currentTarget.value)}
          type='text'
        />
        <button
          disabled={isCreatePending}
          className='comment-button'
        >
          Submit
        </button>
      </div>
    </form>
  );
}

```



# **CommentList.js**

```javascript
import React from 'react';

import Comment from './Comment';

export default function CommentList({ comments }) {

  if (!comments) {

    return null;

  }

  

  return (

    <ul className='comments-list'>

      {

         comments.map((comment)=>{

         return <Comment comment={comment} />})

      }

    </ul>

  );

}
```


# **Comments.js**

```javascript
import React, { useEffect } from 'react';

import { useDispatch, useSelector } from 'react-redux';

import {

  loadCommentsForArticleId,

  selectComments,

  isLoadingComments,

} from '../comments/commentsSlice';

import { selectCurrentArticle } from '../currentArticle/currentArticleSlice';

import CommentList from '../../components/CommentList';

import CommentForm from '../../components/CommentForm';

const Comments = () => {

  const dispatch = useDispatch();

  const article = useSelector(selectCurrentArticle);

  // Declare additional selected data here.

  const comments = useSelector(selectComments);

  const commentsAreLoading = useSelector(isLoadingComments);

  // Dispatch loadCommentsForArticleId with useEffect here.

 useEffect(() => {

    if (article !== undefined) {

      dispatch(loadCommentsForArticleId(article.id))};

  }, [dispatch, article]);

const commentsForArticleId = article ? comments[article.id] : [];

  if (commentsAreLoading) return <div>Loading Comments</div>;

  if (!article) return null;

  return (

    <div className='comments-container'>

      <h3 className='comments-title'>Comments</h3>

      <CommentList comments={commentsForArticleId} />

      <CommentForm articleId={article.id} />

    </div>

  );

};

export default Comments;
```


# **commentSlice.js**

```javascript
// Import createAsyncThunk and createSlice here.
import { createAsyncThunk, createSlice } from '@reduxjs/toolkit';

// Create loadCommentsForArticleId here.
export const loadCommentsForArticleId = createAsyncThunk(
  'comments/loadCommentsForArticleId',
  async(id) => {
    const data = await fetch(`api/articles/${id}/comments`);
    const json = await data.json();
    console.log(json);
    return json;
  }
  
);
// Create postCommentForArticleId here.

export const postCommentForArticleId = createAsyncThunk( 
  'comments/postCommentForArticleId',
  async ({articleId, comment}) => {
    const requestBody = JSON.stringify({comment: comment});
    const response = await fetch(`api/articles/${articleId}/comments`, {
      method: 'POST', 
      body: requestBody
      });
    const json = await response.json();
    return json;
  }
)


export const commentsSlice = createSlice({
  name: 'comments',
  initialState: {
    // Add initial state properties here.
    byArticleId: {},
    isLoadingComments: false,
    failedToLoadComments: false,
    createCommentIsPending: false,
    failedToCreateComment: false,
  },
  // Add extraReducers here.
  extraReducers: (builder) => {
    builder
      .addCase(loadCommentsForArticleId.pending, (state) => {
        state.isLoadingComments = true;
        state.failedToLoadComments = true;
      })
      .addCase(loadCommentsForArticleId.fulfilled, (state, action) => {
        state.isLoadingComments = false;
        state.failedToLoadComments = false;
        state.byArticleId = {[action.payload.articleId]: action.payload.comments}
      })
      .addCase(loadCommentsForArticleId.rejected, (state) => {
        state.isLoadingComments = false;
        state.failedToLoadComments = false;
      })

      .addCase(postCommentForArticleId.pending, (state) => {
        state.createCommentIsPending = true;
        state.failedToCreateComment = false;
      })
      .addCase(postCommentForArticleId.fulfilled, (state, action) => {
        state.createCommentIsPending = false;
        state.failedToCreateComment = false;
        state.byArticleId[action.payload.articleId].push(action.payload)
      })
      .addCase(postCommentForArticleId.rejected, (state) => {
        state.createCommentIsPending = false;
        state.failedToCreateComment = true;
      })
  }
});

export const selectComments = (state) => state.comments.byArticleId;
export const isLoadingComments = (state) => state.comments.isLoadingComments;
export const createCommentIsPending = (state) => state.comments.createCommentIsPending;

export default commentsSlice.reducer;
```

