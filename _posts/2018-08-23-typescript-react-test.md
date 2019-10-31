---
layout: post
title: "React Unit test with Jest, Typescript, Redux"
author: "Tyler Shin"
categories: workflow
tags: [jest, redux, react, test, unit test]
image: packageJson.png
---
# Background
We've been used Jest with [Enzyme](https://github.com/airbnb/enzyme).  
However, because of the decorators(or HoC) we barely unit tests for the React components.  
To do a proper test, I have to mock dependencies, and it's kind of annoying thing and even sometimes it's impossible.  

In the meantime, Jest Provided [Snapshot Testing](https://jestjs.io/docs/en/snapshot-testing) with [react-test-renderer](https://reactjs.org/docs/test-renderer.html).  
Thanks to these tools, I had no reason to use Enzyme and it makes component testing simple.

This document is the simple guide for setting test environment who's  using TypeScript, React, Redux together.  

---

# Common setting
You should set the jest options in the package.json(You can set it with jest.config.js or else. If you want to write config to another file, follow [this guide](https://jestjs.io/docs/en/configuration.html).)  

![package.json setting]({{ site.url }}/assets/img/packageJson.png)

Above is my current config options. we will explore key configs.  

## transform  
This is a pre-processing setting. I used [ts-jest](https://github.com/kulshekhar/ts-jest) to transpile Typescript to Javascript.  You can make your own pre-processing logic like below image.(we had used below pre-processing logic before.)  

![pre-processing setting]({{ site.url }}/assets/img/preProcessing.png)


## moduleNameMapper  
This option is needed to map module to another one. it's similar to module mocking.  
In the above config, we are trying to map asset files(jpg, png, ...) with `fileMock.js`.  
And also map style files(css, scss, less) with [`identity-obj-proxy`](https://github.com/keyanzhang/identity-obj-proxy) package.  

Because TypeScript Compiler can't handle those files, we should mock them.

For example, it makes mock for below code.
```js
const styles = require("./paperItem.scss");
const OPEN_SORTING: require("./open-sorting.svg");
const ORCID_LOGO: "orcid-logo.png";
```

module mocking(avoiding unneeded module files) are rely on [ES6's Proxy feature](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Proxy). If you want to know about the core, I recommend to visit and study about it.  

## setupFiles
the paths to modules that run some code to configure or set up the testing environment before each test.  
(From Jest official docs.)

![setupFiles]({{ site.url }}/assets/img/preload.png)

above settings are needed because jsDOM has an [issue](https://github.com/geelen/react-snapshot/issues/93) with `scrollTo` method.

---

## Common JavaScript(TypeScript) test

![normalUnitTest]({{ site.url }}/assets/img/preload.png)

it's almost the same with typical unit-test. Don't forget unmock your target module file when `autoMock` option is available.  

---

## Action Test
![Redux action file]({{ site.url }}/assets/img/actionFile.png)

Let's assume that we'll make a test for the above action.

In this function, there are two scenarios.  
First, the API call was successful and the Journal data is received.  
Second, the API call was failed.  

So I created the basic structure like below image.  

![action test file]({{ site.url }}/assets/img/actionTest1.png)

### API Mock
First, you should mock the API call for avoiding actual HTTP call.
I've created API mock file under the `__mocks__` directory where the original API file exists.  
(actual file: `/app/api/journal.tsx`, mock file: `/app/api/__mocks__/journal.tsx`)  
it automatically mocks your module with under the same name of `__mocks__`.  
This is called Manual Mock and if you want to know about this more, follow this [link](https://jestjs.io/docs/en/manual-mocks).  

For this time, I made mocked journal API like below image.  

![API mock]({{ site.url }}/assets/img/apiMock.png)

It's simple. if a user asked `0 or false` for the parameter, it throws FAKE ERROR.  
If a user asked numeric normal parameter, it returns journal fixture data. 

(You can mock HTTP itself with [nock](https://github.com/nock/nock) too.)

### Mock Redux Store
The next step is to mock the Redux Store.  
I've used [redux-mock-store](https://github.com/dmitry-zaets/redux-mock-store) to mock store easily.  
The usage is kind of simple. I made the helper function to use it more easily and more scalable.  

```ts
// mockStore.tsx
import thunk from "redux-thunk";
import { AppState } from "../reducers";
import configureStore, { MockStore } from "redux-mock-store";

export const generateMockStore = (state: AppState | {}): MockStore<AppState | {}> => {
  const mockStore = configureStore([thunk]);
  const store = mockStore(state);

  store.clearActions();
  return store;
};
```


### Test hook

```ts
  beforeEach(() => {
    store = generateMockStore({});
    store.clearActions();
  });
```
For every test, we should initialize the redux store for the proper action tracking.  

### Actual Test

```ts
  describe("getJournal action creator", () => {
    describe("when it's succeeded", () => {
      const mockJournalId = 123;

      beforeEach(async () => {
        await store.dispatch(getJournal(mockJournalId));
      });

      it("should return JOURNAL_SHOW_START_TO_GET_JOURNAL type action", () => {
        const actions = store.getActions();

        expect(actions[0]).toEqual({
          type: ACTION_TYPES.JOURNAL_SHOW_START_TO_GET_JOURNAL,
        });
      });

      it("should return GLOBAL_ADD_ENTITY type action", () => {
        const actions = store.getActions();

        expect(actions[1]).toEqual({
          type: ACTION_TYPES.GLOBAL_ADD_ENTITY,
          payload: {
            entities: {
              journals: {
                [`${RAW.JOURNAL.id}`]: RAW.JOURNAL,
              },
            },
            result: 2764552960,
          },
        });
      });

      it("should return PAPER_SHOW_SUCCEEDED_TO_DELETE_COMMENT type action with proper payload", () => {
        const actions = store.getActions();

        expect(actions[2]).toEqual({
          type: ACTION_TYPES.JOURNAL_SHOW_SUCCEEDED_TO_GET_JOURNAL,
          payload: { journalId: RAW.JOURNAL.id },
        });
      });
    });
  });
```
`store.getActions()` returns dispatched actions. So, we can track and test these actions.  


for failure test, let's add failure test too.  

```ts
describe("when it has error", () => {
  const mockJournalId = 0;

  beforeEach(async () => {
    await store.dispatch(getJournal(mockJournalId));
  });

  it("should return JOURNAL_SHOW_START_TO_GET_JOURNAL type action", () => {
    const actions = store.getActions();

    expect(actions[0]).toEqual({
      type: ACTION_TYPES.JOURNAL_SHOW_START_TO_GET_JOURNAL,
    });
  });

  it("should return JOURNAL_SHOW_FAILED_TO_GET_JOURNAL type action", () => {
    const actions = store.getActions();

    expect(actions[1]).toEqual({
      type: ACTION_TYPES.JOURNAL_SHOW_FAILED_TO_GET_JOURNAL,
    });
  });
});
```

---

## Container Component Test

Let's assume we have a container component. our example code is [here](https://github.com/pluto-net/scinapse-web-client/blob/master/app/components/journalShow/index.tsx). I omitted this code because it's too long.  

At first, we will use [react-test-renderer](https://reactjs.org/docs/test-renderer.html) rather than Enzyme.  
If you want to test user interactions(click, type, etc...), [Enzyme](https://github.com/airbnb/enzyme) could be a great option.  

below is our test code.  

```ts
jest.mock("../../../api/journal");

import * as React from "react";
import * as renderer from "react-test-renderer";
import { MemoryRouter, Route } from "react-router";
import { Provider } from "react-redux";
import { generateMockStore } from "../../../__tests__/mockStore";
import { initialState } from "../../../reducers";
import JournalShowContainer from "..";
import { JOURNAL_SHOW_PATH } from "../../../routes";
import { RAW } from "../../../__mocks__";

describe("JournalShow Container Component", () => {
  let mockStore = generateMockStore(initialState);

  beforeEach(() => {
    mockStore.clearActions();
  });

  describe("when journal data exist", () => {
    beforeEach(async () => {
      const journalPaper = RAW.JOURNAL_PAPERS_RESPONSE.data.content[0];

      const mockState = {
        ...initialState,
        journalShow: {
          ...initialState.journalShow,
          journalId: RAW.JOURNAL.id,
          paperCurrentPage: 1,
          paperIds: [journalPaper.id],
          paperTotalPage: 1,
        },
        entities: {
          ...initialState.entities,
          journals: { [`${RAW.JOURNAL.id}`]: RAW.JOURNAL },
          papers: { [`${journalPaper.id}`]: journalPaper },
        },
      };

      mockStore = generateMockStore(mockState);
    });

    it("should render correctly", () => {
      const tree = renderer
        .create(
          <Provider store={mockStore}>
            <MemoryRouter initialIndex={0} initialEntries={["/journals/2764552960"]}>
              <Route path={JOURNAL_SHOW_PATH}>
                {/*Route is needed for providing match params*/}
                <JournalShowContainer />
              </Route>
            </MemoryRouter>
          </Provider>
        )
        .toJSON();
      expect(tree).toMatchSnapshot();
    });
  });
});
```

First, we need to create mockStore with app initial state.  

```ts
let mockStore = generateMockStore(initialState);
```

above one is that code. In this code, initialState means my app's actual redux initialState.

```ts
// initialState
export const initialState: AppState = {
  configuration: ConfigurationReducer.CONFIGURATION_INITIAL_STATE,
  signUp: signUpReducer.SIGN_UP_INITIAL_STATE,
  signIn: signInReducer.SIGN_IN_INITIAL_STATE,
  dialog: dialogReducer.DIALOG_INITIAL_STATE,
  home: HOME_INITIAL_STATE,
  layout: LAYOUT_INITIAL_STATE,
  emailVerification: emailVerificationReducer.EMAIL_VERIFICATION_INITIAL_STATE,
  currentUser: CURRENT_USER_INITIAL_STATE,
  articleSearch: ARTICLE_SEARCH_INITIAL_STATE,
  paperShow: PAPER_SHOW_INITIAL_STATE,
  authorShow: AUTHOR_SHOW_INITIAL_STATE,
  journalShow: JOURNAL_SHOW_INITIAL_STATE,
  collectionShow: INITIAL_COLLECTION_SHOW_STATE,
  userCollections: USER_COLLECTIONS_INITIAL_STATE,
  entities: INITIAL_ENTITY_STATE,
};
```

And we should modify our initial state and re-generate store with the modified state for the proper test condition.  
(in this case, journal and paper data are needed to render content.)
So, we make the mockState and mockStore. Then, Trying to make the snapshot with `react-test-renderer`'s renderer.  

In this step, You should wrap the target container component with `<Provider />` component and `<MemoryRouter />` component(If you're using React-Router-V4).  

These are why they're needed for.
- `<Provider />` component will inject our Redux logic into React context. (you can pass our mockStore which applying mockState here.)
- `<MemoryRouter />` will pass our Location data into React context. it needed for `<Link />` component in React Router v4.
- `<Route path={JOURNAL_SHOW_PATH}>` is needed for providing match params.

If the target component is a dumb component and doesn't have any dependency with Redux & React Router, you can omit all of these wrapper components.  

that's it!

`react-test-renderer` will make the snapshot of your target container component and jest will test this snapshot more.  

---

## Reducer Test
![Reducer Spec]({{ site.url }}/assets/img/reducerSpec.png)

It's pretty easy now. just mock the target state and target action.  
Then you can easily make the unit test like the above one.
