import * as anchor from '@coral-xyz/anchor';
import { Program } from '@coral-xyz/anchor';
import { Keypair, PublicKey } from '@solana/web3.js';
//import { Votingdapp } from '@/../anchor/target/types/votingdapp';
import { Votingdapp } from '../target/types/votingdapp';
//import { BankrunProvider, startAnchor } from 'anchor-bankrun';
import { startAnchor } from 'solana-bankrun';
import { BankrunProvider } from 'anchor-bankrun';

const IDL = require('../target/idl/votingdapp.json');

/*const votingdappAddress = new PublicKey(
  'AsjZ3kWAUSQRNt2pZVeJkywhZ6gpLpHZmJjduPmKZDZZ'
);*/
const votingdappAddress = new PublicKey(
  'ARCa2n7p7CFanHpEDhygY5i9o1Jbqwr87qmFkncTJQ49'
);

describe('votingdapp', () => {
  let context;
  let provider;
  //anchor.setProvider(anchor.AnchorProvider.env());
  let votingdappProgram: Program<Votingdapp>;
  //let votingdappProgram = anchor.workspace.Votingdapp as Program<Votingdapp>;

  beforeAll(async () => {
    context = await startAnchor("", [{ name: "votingdapp", programId: votingdappAddress }], []);
    provider = new BankrunProvider(context);
    votingdappProgram = new Program<Votingdapp>(
      IDL,
      provider,
    )
  })
  // Test 1:
  it('Initialize Poll', async () => {
    await votingdappProgram.methods
      .initializePoll(
        new anchor.BN(1),
        'What is your favorite type of peanut butter',
        new anchor.BN(0),
        new anchor.BN(1821246680),
      )
      .rpc();

    const [pollAddress] = PublicKey.findProgramAddressSync(
      // Grab the buffer from a u64
      [new anchor.BN(1).toArrayLike(Buffer, 'le', 8)],
      votingdappAddress // our Votingdapp program address
    );

    const poll = await votingdappProgram.account.poll.fetch(pollAddress); // Think of poll as our direct access to data inside PollAccount' account

    console.log(poll);

    expect(poll.pollId.toNumber()).toEqual(1);
    expect(poll.description).toEqual('What is your favorite type of peanut butter');
    expect(poll.pollStart.toNumber()).toBeLessThan(poll.pollEnd.toNumber());
  });

  // Test 2:
  it('Initialize Candidate', async () => {
    await votingdappProgram.methods
      .initializeCandidate('Smooth', new anchor.BN(1))
      .rpc();
    await votingdappProgram.methods
      .initializeCandidate('Crunchy', new anchor.BN(1))
      .rpc();

    const [crunchyAddress] = PublicKey.findProgramAddressSync(
      [new anchor.BN(1).toArrayLike(Buffer, 'le', 8), Buffer.from('Crunchy')],
      votingdappAddress,
    );
    /*const [crunchyAddress] = PublicKey.findProgramAddressSync(
      [Buffer.from('Crunchy'), new anchor.BN(1).toArrayLike(Buffer, 'le', 8),],
      votingdappAddress,
    );*/
    const crunchyCandidate = await votingdappProgram.account.candidate.fetch(crunchyAddress);
    console.log(crunchyCandidate);
    expect(crunchyCandidate.candidateVotes.toNumber()).toEqual(0);

    const [smoothAddress] = PublicKey.findProgramAddressSync(
      [new anchor.BN(1).toArrayLike(Buffer, 'le', 8), Buffer.from('Smooth')],
      votingdappAddress,
    );
    /*const [smoothAddress] = PublicKey.findProgramAddressSync(
      [Buffer.from('Smooth'), new anchor.BN(1).toArrayLike(Buffer, 'le', 8),],
      votingdappAddress,
    );*/
    const smoothCandidate = await votingdappProgram.account.candidate.fetch(smoothAddress);
    console.log(smoothCandidate);
    expect(smoothCandidate.candidateVotes.toNumber()).toEqual(0);
  });

  // Test 3:
  it('vote', async () => {
    await votingdappProgram.methods
      .vote(
        "Smooth",
        new anchor.BN(1)
      )
      .rpc();

    const [smoothAddress] = PublicKey.findProgramAddressSync(
      [new anchor.BN(1).toArrayLike(Buffer, 'le', 8), Buffer.from('Smooth')],
      votingdappAddress,
    );
    /*const [smoothAddress] = PublicKey.findProgramAddressSync(
      [Buffer.from('Smooth'), new anchor.BN(1).toArrayLike(Buffer, 'le', 8),],
      votingdappAddress,
    );*/
    const smoothCandidate = await votingdappProgram.account.candidate.fetch(smoothAddress);
    console.log(smoothCandidate);
    expect(smoothCandidate.candidateVotes.toNumber()).toEqual(1);
  });
});