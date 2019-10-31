---
layout: post
title: "How to apply Normalizr with TypeScript in Redux structure"
author: "Tyler Shin"
categories: frontend
tags: [React, Redux, Typescript, ImmutableJS, ReturnType]
image: typeWithAction.png
---

## Background

I've been concerned about the 'data' structure of Redux state for a long time.  
As a result, I tried to use 'ImmutableJS' and other tools for 'immutability' of javascript object.

However, I've inspired by [Young](https://youngk.im/) at [Vingle](https://medium.com/vingle-tech-blog) TechTalk about this topic.

They had very similar Redux structure with [SciNapse](https://scinapse.io. but after I left the team, they changed many things in a good way.

Because of Typescript and ImmutableJS and other fancy things, our project was too complicated. ?For hiring more developer and making our project as an open source project, I should reduce our complexity as possible as I can.

So, I decided to follow and improve Vingle's changes.

## Technical Background

**BEFORE**

* React
* Redux
* Typescript
* ImmutableJS

**AFTER**

* ~~ImmutableJS~~
* [Normalizr](https://github.com/paularmstrong/normalizr)

The whole migration is saparated by two parts.  
One, Use Typescript's features rather than ImmutableJS. (Read-only, Conditional Types, ...)  
Two, Use Normalizr for data clean up and organize.

## Implementation

### PLAN

Since scinapse is already large enough and complicated, I thought it was impossible to change all the code at once.  
So I tried to apply new logics to new component and features and let existing code works fine.

### Read-Only

TypeScript has Readonly type definition as default.  
you can find definition and example from [here](https://basarat.gitbooks.io/typescript/docs/types/readonly.html).  
We can apply this rather than immutableJS.

example)

```ts
// BEFORE
export interface AuthorShowState {
  papers: List<PaperRecord>;
  authorId: number | null;
  coAuthorIds: number[];
  papersTotalPage: number;
  papersCurrentPage: number;
  papersSort: AUTHOR_PAPERS_SORT_TYPES;
}

// AFTER
export interface AuthorShowState
  extends Readonly<{
      paperIds: number[];
      authorId: number | null;
      coAuthorIds: number[];
      papersTotalPage: number;
      papersCurrentPage: number;
      papersSort: AUTHOR_PAPERS_SORT_TYPES;
    }> {}
```

### Conditional Type and Return Type in Typescript

You should understand about Conditional Type and Return Type in Typescript.  
The [official update log](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-8.html) explains about this well.  
I strongly recommend read the above article first.

Since there is no reason to re-describe the duplicate content, I'll omit the specific description and just give example code. If you read above article, it will be easy to understand.

```ts
export function createAction<T extends { type: ACTION_TYPES }>(d: T): T {
  return d;
}

export const ActionCreators = {
  addEntity(payload: { entities: { [K in keyof AppEntities]?: AppEntities[K] }; result: number | number[] }) {
    return createAction({ type: ACTION_TYPES.GLOBAL_ADD_ENTITY, payload });
  },

  flushEntities() {
    return createAction({ type: ACTION_TYPES.GLOBAL_FLUSH_ENTITIES });
  },
};
export type ActionUnion<T extends ActionCreatorsMapObject> = ReturnType<T[keyof T]>;

export type Actions = ActionUnion<typeof ActionCreators>;
```

You can apply "Actions" type to each reducer function.  
The example is in the next chapter.

### Entity Reducer with Normalize

Entity reducer is a collection of all data used in an web app.  
As a result, each component retrieves the necessary data from the entity reducer through the mapStateToProps function.

First of all, we should get data from API server and normalize it with Normalizr.  
I'll handle this logic at the API file.

```ts
class AuthorAPI extends PlutoAxios {
  public async getAuthor(
    authorId: number,
  ): Promise<{
    entities: { authors: { [authorId: number]: Author } };
    result: number;
  }> {
    const res = await this.get(`/authors/${authorId}`);
    const rawAuthor: RawAuthorResponse = res.data.data;

    const normalizedData = normalize(
      {
        id: rawAuthor.id,
        name: rawAuthor.name,
        hIndex: rawAuthor.hindex,
        lastKnownAffiliation: rawAuthor.last_known_affiliation,
        paperCount: rawAuthor.paper_count,
        citationCount: rawAuthor.citation_count,
      },
      authorSchema,
    );
    return normalizedData;
  }
}
```

The actual action is fired like below.

```ts
// actions.tsx
export function getAuthor(authorId: number) {
  return async (dispatch: Dispatch<any>) => {
    try {
      const authorResponse = await AuthorAPI.getAuthor(authorId);

      /* LOOK AT THIS */
      dispatch(ActionCreators.addEntity(authorResponse));
      /* LOOK AT THIS */
      dispatch(ActionCreators.getAuthor({ authorId: authorResponse.result }));
    } catch (err) {
      alertToast({
        type: "error",
        message: "Failed to get author information",
      });
    }
  };
}
```

Entity reducer looks like below.

```ts
export type AppEntities = {
  authors: {
    [authorId: number]: Author;
  };
  papers: {
    [paperId: number]: Paper;
  };
};

export interface EntityState extends Readonly<AppEntities> {}

export const INITIAL_ENTITY_STATE = {
  authors: {},
  papers: {},
};

// Actions type we declared before is used here.
export function reducer(state: EntityState = INITIAL_ENTITY_STATE, action: Actions) {
  switch (action.type) {
    case ACTION_TYPES.GLOBAL_ADD_ENTITY:
      const { entities } = action.payload;

      if (!entities) {
        return state;
      }

      return {
        ...state,
        authors: { ...state.authors, ...entities.authors },
        papers: { ...state.papers, ...entities.papers },
      };

    case ACTION_TYPES.GLOBAL_FLUSH_ENTITIES:
      return INITIAL_ENTITY_STATE;

    default:
      return state;
  }
}
```

Then you can get type of action's payload at the inside of the entity reducer's switch statement.

AppEntities type defines all dataset what we will use.  
The reducer function handling the entities. The important thing is the spread operator works only shallow level.(1 depth only)  
So, you should do extra type definition and reducer handling for multi-depth entity change.  
However, until now I have not needed the multi-depth data change yet.

## Conclusion

### Benefit

* Reduce complexity of the project.
* Get rid of ImmutableJS. (well, it'll be tough...)
* Strict type bundling between action and reducer and actionTypes.

### Limit

There are too many things to change for now, I'll be busy...sad...
