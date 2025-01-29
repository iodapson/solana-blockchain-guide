```ts
import * as anchor from '@coral-xyz/anchor';
import { Program } from '@coral-xyz/anchor';
import { Keypair, PublicKey } from '@solana/web3.js';
import { Votingdapp } from '../target/types/votingdapp';
import { BankrunProvider, startAnchor } from 'anchor-bankrun';
//import { startAnchor } from 'solana-bankrun';
//import { BankrunProvider } from 'anchor-bankrun';

const IDL = require('../target/idl/votingdapp.json');

const votingdappAddress = new PublicKey(
	'AsjZ3kWAUSQRNt2pZVeJkywhZ6gpLpHZmJjduPmKZDZZ'
);

describe('votingdapp', () => {
	it('Initialize Poll', async () => {
		const context = await startAnchor(
			'',
			[{ name: 'votingdapp', programId: votingdappAddress }],
			[]
		);
		const provider = new BankrunProvider(context);

		const votingdappProgram = new Program<Votingdapp>(IDL, provider);

		await votingdappProgram.methods
			.initializePoll(
				new anchor.BN(1),
				'What is your favorite type of peanut butter',
				new anchor.BN(0),
				new anchor.BN(1821246680)
			)
			.rpc();

		const [pollAddress] = PublicKey.findProgramAddressSync(
			// Grab the buffer from a u64
			[new anchor.BN(1).toArrayLike(Buffer, 'le', 8)],
			votingdappAddress // our Votingdapp program address
		);

		const poll = await votingdappProgram.account.poll.fetch(pollAddress); // Think of poll has our direct access to data inside PollAccount' account

		console.log(poll);

		expect(poll.pollId.toNumber()).toEqual(1);
		expect(poll.description).toEqual(
			'What is your favorite type of peanut butter'
		);
		expect(poll.pollStart.toNumber()).toBeLessThan(poll.pollEnd.toNumber());
	});
});
```
