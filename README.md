# Temple-Of-Juno-SmartContract
Rust smart contract application deployed on Juno testnet. Will be deployed on Juno mainnet after go-live on Oct 1 2021.

The purpose of this project is to create a Rust smart contract on CosmWasm with JavaScript frontend that allows for a polling system, with all polls, votes, and voters to be recorded on the blockchain. This project will be deployed as part of Juno testnet and will go live when Juno mainnet goes live.


Release functionality:

* Users have the ability to connect their Keplr wallet with the frontend application. This is a requirement to perform all other actions, as all other actions require $JUNO Keplr transactions.
* Users have the ability to create polls with default or customized poll responses by signing a transaction. Polls and their poll response options are permanent and cannot be changed.
* Users have the ability to upvote or downvote on polls by signing a transaction; this is synonymous with the "popularity" of the poll. Upvote/downvote on polls can be changed with a new transaction.
* Users have the ability to vote on a poll for a specific poll answer choice by signing a transaction. The voted choice of polls can be changed with a new transaction.

Stretch goal functionality:
* Users have the ability to tip the asker of a poll by signing a transaction and specifying the amount of $JUNO they would like to tip.


Dependencies:

Any client that wants to use the below actions MUST have their Keplr wallet connected to the app. 
If the Keplr wallet is not connected, the buttons that allow for these actions will not appear and they cannot be submitted. 


Breakdown of User Actions:

- Connect Keplr wallet
  - JavaScript Side:
    - This is the default/first question that is asked of the user. The webpage will be blank except for the logos and 
      the prompt to ask the user to connect their Keplr wallet.
    - When user hits connect, JavaScript code will connect this session with the user's Keplr wallet.
    - This looks like sample JS code to connect to Keplr: https://github.com/ebaker/next-cosmwasm-keplr-starter/blob/5b6d0437c5097faef90ccacfa502e14a65d82c00/services/keplr.tsx#L11

- View polls
  - JavaScript Side:
    - If user has connected Keplr wallet, this is the default screen that will be shown.
    - The most recent X polls can have their IDs retrieved by calling the smart contract's get_recent_polls(X) method.
    - The data for each pollID can be pulled by calling the smart contract methods:
      - get_poll_type(pollID)
      - get_poll_question(pollID)
      - get_poll_question_upvotes_downvotes(pollID)    // to display graphically next to upvote/downvote buttons
      - get_poll_answer_choices(pollID)
    - Next to each poll is a pie graph of the poll's results. The voted answers can be pulled by the smart contract method:
      - get_poll_voted_answers(pollID)

- Upvote or Downvote poll question
  - JavaScript Side:
    - Each poll has an up arrow and down arrow next to the poll on the left (similar to StackOverflow)
    - The user can upvote or downvote a poll. This represents the "popularity" of the poll.
    - Either above or inside the arrows should be a "total poll score", which is upvotes minus downvotes.
    - The upvote or downvote button will show as highlighted if the user has already voted on the poll
    - If the user clicks either upvote or downvote buttons for the first time on a poll, it will signal a transaction to the smart contract.
    - If the user has already upvoted or downvoted on a poll, only a change in previous voting will signal a transaction to the smart contract.
    - The user will be prompted with the Keplr prompt to sign the transaction of upvote or downvote.
    - Once the transaction goes through asynchronously, the poll value will update to the new poll's value.
  - Smart Contract side: 
    - When the user submits the transaction on the poll, JS will call upvote_poll or downvote_poll with the ID.
    - Send transaction confirmation to user 
    - Check to confirm that user has not previously submitted same vote on same poll question; if so, return error and reverse transaction.

- Create simple poll
  - JavaScript Side:
    - User selects "create simple poll" button
    Popup appears with the following information:
      - Test input field for the question to be asked in the poll
      - Yes/No button if user wants to ask anonymously
      - Text containing the default answer choices for the poll will be {Yes, No, No with Veto, Abstain}
      - Two buttons at bottom of popup: {Submit Poll, Cancel}
      - Hitting submit calls the smart contract's create_simple_poll() with the question and if the user wanted to be anonymous.
      - There is validation on JS side before submit can be pressed that question text does not contain special characters aside from (,),$,@,!,.,,,%. No quotes allowed.
      - User is prompted with a transaction sign to confirm poll is placed on blockchain.
      - If create_simple_polls() returns Error, need an error notification to display on screen with reason.
  - Smart Contract side:
    - calls create_simple_polls
    - generates random ID
    - assign creator's address to [creator], unless specified anonymous then None
    - creates [Question] (cannot be changed), initializes [QuestionUpvotes/QuestionDownvotes] to 0
    - creates [Options] (cannot be changed), initializes [OptionsVotes] to all be 0
    - Submits transaction to user before Poll officially created and added to [Polls]
    - If transaction not success, return Error.
    - add Poll to [Polls]
    - Return Error if any part failed, or new ID for poll created if success.

- Create multi poll
  - JavaScript Side:
    - User selects "create multi poll" button
    Popup appears with the following information:
      - Test input field for the question to be asked in the poll
      - Yes/No button if user wants to ask anonymously
      - Input fields with custom user input text for up to 5 responses to the poll
      - Two buttons at bottom of popup: {Submit Poll, Cancel}
      - Hitting submit calls the smart contract's create_multi_poll() with the question, answer choices, and if the user wanted to be anonymous.
      - There is validation on JS side before submit can be pressed that question and answer option texts do not contain special characters aside from (,),$,@,!,.,,,%. No quotes allowed.
      - User is prompted with a transaction sign to confirm poll is placed on blockchain.
      - If create_multi_polls() returns Error, need an error notification to display on screen with reason.
  - Smart Contract side:
    - calls create_multi_polls
    - generates random ID
    - assign creator's address to [creator], unless specified anonymous then None
    - creates [Question] (cannot be changed), initializes [QuestionUpvotes/QuestionDownvotes] to 0
    - assigns [Options] to provided options (cannot be changed), initializes [OptionsVotes] to all be 0
    - Submits transaction to user before Poll officially created and added to [Polls]
    - If transaction not success, return Error.
    - add Poll to [Polls]
    - Return Error if any part failed, or new ID for poll created if success.

