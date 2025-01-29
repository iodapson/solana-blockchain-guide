<h3>Add Blinks and Actions to'votingdapp' Developed Earlier</h3>

#### Preparing the 'votingdapp' project for adding Blinks and Actions:

On this page, we'll add Blinks and Actions to our previously created `votingdapp` project, and there is a lot of different amount of work that we'd have to do for Blinks and Actions. We will be referencing the [Solana Actions and Blinks documentation page](https://solana.com/docs/advanced/actions) to prepare our `votingdapp` project for adding Blinks and Actions to it.

<span style="color: orange;">1</span>

#### Deploy your dapp (e.g, 'votingdapp'):

In the previous chapter ('Project 2: Voting Application'), we used `anchor-bankrun` to run tests in the [Testing Dapps](10.%20Testing%20Dapps.md) page. However, in this chapter, we will need the local test validator to test our dapp that has some UI bootstrap on top of it.

Let us first go into our `anchor` folder and perfom an Anchor deploy operation. Now this might be different based off your Solana config, so actually, you might need to check your Solana config first with - `$ solana config get`. We need to set our config environment to local (RPC URL: http://localhost:8999).

Set your Solana config environment to local if it isn't already set to local:

```bash
solana config set -ul
```

Now we can run the anchor deployment command:

```bash
anchor deploy
```

The above command will deploy your dapp project's binary, e.g, `votingdapp.so`. to the blockchain.

If `$: anchor deploy` fails to work with an error message <span style="color: #722f37; font-size: 12; font-weight: 600;">error sending request for url (http://127.0.01;8099): error trying to connect: tcp connect error: Connection refused (os error 111)</span>, then it means that your 'solana-test-validator' is not running. You can fix the error by just running `$: solana-test-validator` to start the 'solana-test-validator' which is a requirement to deploy your anchor smart contract locally. Upon starting the 'solana-test-validator', retry the `$: anchor deploy` command.

Check your successfully deployed program at [https://explorer.solana.com/](https://explorer.solana.com/) - switch from `Mainnet Beta` to `Custom RPC URL`, then copy, paste, and enter the `Program Id` displayed on your terminal into the search bar that has a placeholder text that says - <small style="font-size: 14.5px; letter-spacing: 0.25px;">Search for blocks, accounts, transactions, programs, and tokens</small>.

After successfully deploying your dapp to the local validator (using `solana-test-validator`), and searching for your dapps's `Program Id` on [explorer.solana.com](explorer.solana.com), you should see something somewhat similar to the following:

![Screenshot of explorer.solana.com votingdapp Program Id search](./assets/explorer.solana.com-votingdapp-Program-Id-search-result.png 'Result of the search for our votingdapp Program Id on Solana Explorer')

<span style="color: orange;">Good to know 1:</span>

The `solana-test-validator` command when run for the first time, creates a new folder called `test-ledger` inside the working directory where the command is run. If you delete this folder and re-run the command, the folder will simply be recreated.

<br/>

<span style="color: orange;">2</span>

#### Install @solana/actions:

Go into your project's root directory. If you currently inside the `anchor` directory, then you have to `$ cd ..` so you can be inside your project's root directory. Use the command below to install `@solana/actions`:

NPM:

```bash
npm install @solana/actions
```

<br/>

<span style="color: orange;">3</span>

#### Run your dapp

Let us now run our `votingdapp` in dev mode:

```
npm run dev
```

<br/>

<span style="color: orange;">4</span>

#### Create a GET request to get the right information to do our Actions:

We need to structure our request in such a way that follows the guidelines specified in the Solana Actions documentation page' URL schema here - [https://solana.com/docs/advanced/actions#url-scheme](https://solana.com/docs/advanced/actions#url-scheme).

- We will write the URL Schema for our Actions inside a `route.ts` code-file located at - `web` - `app` - `api`. For you, the right folder may be `src` - `app` - `api`. Create a new sub-folder named `votingdapp` inside the `api` folder. Please note that in the bootcamp's video, the new folder was named `voting` instead of `votingdapp`.

- Now copy the `route.ts` code file inside `.../api/hello` into our previous created new `.../api/votingdapp` sub-folder. The copied `route.ts` files should have the following content:

```ts
export async function GET(request: Request) {
	return new Response('Hello, from API!');
}
```

- You can check out the newly created route by opening `localhost:3000/api/votingdapp`. You will find the text `Hello, from API!` written on your screen.

<span style="color: orange;">Good to know 2:</span>

The template we have chosen to build our dapp from spins up a basic NextJS server-side app for working with blinks, which should be noted to be quite a heavy application for these blinks. NextJS server side is quite heavy for this for this usage. It is recommended to use a Node JS server - like an Express server if you can. But because we have a scaffold and we will later on in the course build the actual frontend for the voting application, we are just going to use this backend and bootstrap it with this voting Action.

<br/>

<span style="color: orange;">5</span>

#### Return valuable information in order to display the blink on any social media

The expected metadata response after an Action client makes a GET request to an Action provider for its available Actions is a JSON body-payload that follows the specification of `ActionGetResponse`:

```ts
export type ActionType = 'action' | 'completed';

export type ActionGetResponse = Action<'action'>;

export interface Action<T extends ActionType> {
	/** type of Action to present to the user */
	type: T;
	/** image url that represents the source of the action request */
	icon: string;
	/** describes the source of the action request */
	title: string;
	/** brief summary of the action to be performed */
	description: string;
	/** button text rendered to the user */
	label: string;
	/** UI state for the button being rendered to the user */
	disabled?: boolean;
	links?: {
		/** list of related Actions a user could perform */
		actions: LinkedAction[];
	};
	/** non-fatal error message to be displayed to the user */
	error?: ActionError;
}
```

Fields `icon`, `title`, `description`, and `label` are required. The remaining fields are optional. We will also need to provide CORS information. For now our `api/votingdapp`'s `route.ts` now looks like this:

```ts
import { ActionGetResponse } from '@solana/actions';

export async function GET(request: Request) {
	const actionMetadata: ActionGetResponse = {
		icon: 'https://zestfulkitchen.com/wp-content/uploads/2021/09/Peanut-butter_hero_for-web-2.jpg',
		title: 'Vote for your favorite type of peanut butter!',
		description: 'Vote between crunchy and smooth peanut butter',
		label: 'Vote',
	};
	return Response.json(actionMetadata);
}
```

<span style="color: orange;">Good to know 3:</span>

You can test your blinks using the [dial.to](dial.to) website. To use `dial.to` to test your blinks, you'd include the localhost link to your blink within the `dial.to` route, similar to this - `https://dial.to/?action=solana-action:http://localhost:3000/api/<your-blink-here>`. Make sure to avoid CORS related errors. Read the [step 6](#check-out-your-blink-visually-using-dialto-and-fix-cors-errors) to know exactly how to prevent CORS-related errors that prevent your blinks from loading when testing blinks.

<br/>

<span style="color: orange;">6</span>

#### Check out your blink visually using dial.to, and fix CORS errors

In our case, we can use the `dial.to` website to have a visual representation of what our blink looks like by using the this specific link - `https://dial.to/?action=solana-action:http://localhost:3000/api/votingdapp`. Upon visiting this link, you'd realise that the blink actually fails to load and is thus, blank. That's because we are yet to fix CORS (Cross Origin Resource Sharing) requirement that your browser has in place to keep you safe online.

![dial.to Blink CORS error screenshot](./assets/dial.to-blink-cors-error-screenshot.png 'https://dial.to blink CORS error')

Let's fix the error by modifying our `api/votingdapp/route.ts` file:

```ts
/* route.ts */

import { ActionGetResponse } from '@solana/actions';

export async function GET(request: Request) {
	const actionMetadata: ActionGetResponse = {
		icon: 'https://zestfulkitchen.com/wp-content/uploads/2021/09/Peanut-butter_hero_for-web-2.jpg',
		title: 'Vote for your favorite type of peanut butter!',
		description: 'Vote between crunchy and smooth peanut butter',
		label: 'Vote',
	};
	return Response.json(actionMetadata);
}
```

After making the above modifications, our Action can now be served by a blink, and if you check our the link to our blink using the appropriate `dial.to` link to our blink, our blink should now load.

We get a warning that your blink is not yet registered, and that's okay for us since we are just testing to make sure it works how we want it to for now. The warning is there to let you know blinks that are known by a blink registry. Using an unknown blink is done at your own risk:

![Loaded but partially finished blink](./assets/loaded-but-partially-finished-blink.png 'Dial.to warning that our blink is yet to be registered')

Alright, so now we at least have our blink on display using `dial.to`, but our blink can't post a voting Action. If you click the vote button, you will in fact get a CORS and POST network error.

<span style="color: orange;">Good to know 4:</span>

Part of the whole system for onboarding blinks as of the time of the bootcamp 2024, onto social-media, is that you have to actually get your Action registered via some kind of registry out there, e.g. Dialect.

<br/>

<span style="color: orange;">7</span>

#### Respond with metadata and available Actions

Let's first prevent a CORS error when making a request inside our blink e.g cliking the `vote` button to vote inside our `votingdapp` blink rendered with `dial.to`.

```ts
/*
  src/app/api/votingdapp/route.ts
*/

// imports
...

export const OPTIONS = GET

// export async function GET(request: Request)
...

```

Next, we have to update the `route.ts` for our Blink with the correct `LinkedAction` metadata so we can showcase corresponding available Actions inside the blink. The `ActionGetResponse` [spec](#return-valuable-information-in-order-to-display-the-blink-on-any-social-media) allows us to specify a `LinkedAction` value to its `links?` attribute which is what specifies a list of related Actions for a Blink user to possibly perform. For more details about `LinkedActions` spec, check [this page of Solana docs](https://solana.com/docs/advanced/actions#get-response-body).

And since in our `votingdapp`, we would be performing one of two different post Actions to vote either Crunchy or Smooth, our `ActionGetResponse`' `links` field is specified as:

```ts
...

links: {
    actions: [
        {
            label: "Vote for Crunchy",
            href: "/api/vote?candidate=crunchy",
            type: "post",
        },
        {
            label: "Vote for Smooth",
            href: "/api/vote?candidate=smooth",
            type: "post",
        }
    ]
}
```

The `route.ts` for our `votingdapp` API now lookS like this after specifying a `LinkedAction` value for our `ActionGetResponse`' (GET Response body)`links?:` field:

```ts
/* 
  src/app/api/votingdapp/route.ts 
*/

import { ActionGetResponse, ACTIONS_CORS_HEADERS } from '@solana/actions';

export const OPTIONS = GET;

export async function GET(request: Request) {
	const actionMetadata: ActionGetResponse = {
		icon: 'https://zestfulkitchen.com/wp-content/uploads/2021/09/Peanut-butter_hero_for-web-2.jpg',
		title: 'Vote for your favorite type of peanut butter!',
		description: 'Vote between crunchy and smooth peanut butter',
		label: 'Vote',
		links: {
			actions: [
				{
					label: 'Vote for Crunchy',
					href: '/api/vote?candidate=crunchy',
					type: 'post',
				},
				{
					label: 'Vote for Smooth',
					href: '/api/vote?candidate=smooth',
					type: 'post',
				},
			],
		},
	};
	return Response.json(actionMetadata, { headers: ACTIONS_CORS_HEADERS });
}
```

<br/>

<span style="color: orange;">8</span>

#### Make a 'POST' request with the blink user's wallet pubkey

Let's now create a new function called `POST` for this purpose and learn what we need to include for our `votingdapp` to work.

1. Inside this function, we will need to grab the URL from the request, and the search parameter inside the same URL.

```ts
 // src/app/api/votingdapp/route.ts

...

export async function POST(request: Request) {
    const url = new URL(request.url);
    const candidate = url.searchParams.get("candidate");
}

```

2. In our `votingdapp`, we need to make sure that the vote-candidate given is a valid candidate - 'Crunchy' or 'Smooth' without giving the user of the Blink a strange error in case someone enter a non-existing candidate.

```ts
// src/app/api/votingdapp/route.ts

// imports
...

export async function POST(request: Request) {
    const url = new URL(request.url);
    const candidatae = url.searchParams.get("candidate");

    if (candidate != "crunchy" && candidate != "smooth") {
        return new Response("Invalid candidate", { status: 400 });
    }
}

```

3. Create the transaction to send back after the POST request completes. In order to create transactions, we need a few things - a blockhash, a message, and any signatures that we possibly need to send to be added to that transaction.

Fortunately for us in our `votingdapp` case though, we don't need any signatures on the server side because we don't care about signing anything on the server-side.

WHAT IS THE CONNECTION FOR?

You need to provide a commitment status when you create a connection inside the POST function:

```ts
const connection = new Connection('http://127.0.0.1:8899', 'confirmed');
```

<span style="color: orange;">Good to know 4:</span>

There are various commitment status levels such as; "process", "confirmed", "finalized". And they tell us how certain it is that a transaction would make it on the cluster. Commitment status level `process` says - hey, I found it, it's not yet confirmed. Commitment status level `confirmed` means - it's confirmed by 66% of the actual cluster. And then `finalized` says - it's confirmed by 66% and there's been 31 blocks afterwards that were also confirmed. `finalized` takes a lot longer, `confirmed` is generally safe - at the time of the bootcamp's recording, there's never been something that was confirmed that didn't make it to finalized.

<span style="color: orange;">Side Note:</span>

Moving forward with this guidebook, it is assumed that you now have a Solana wallet. This guidebook makes use of the phantom wallet. Please go to the official Phantom website - [https://phantom.com](https://phantom.com), and install the Phantom extention on your favorite browser(s). This guide also assumes that you created a `seed phrase wallet`.

4. Alright, so next, we will grab all the information that we need from the URL (connection we just created). The information we need are;

- The account that the user signed with on the Blink in order to create the transaction successfully. We would need to copy the user account information from the body of the POST request in our blink - e.g, in the POST request to vote for "Crunchy". In our case, it is the pubkey tied to your wallet:

![POST payload to vote a candidate e.g Crunchy](./assets/POST-payload-to-vote-a-candidate-e.g-Crunchy.png 'Crunchy vote Request Payload')

```ts
const body: ActionPostRequest = await request.json();
const voter = new PublicKey(body.account);
```

---

Timestamp - [1:51:56 - ]
