# AuctionSmartContractTests_

# Auction Smart Contract Tests

This repository contains unit tests and integration tests for an Auction Smart Contract, implemented using the Hardhat framework. The tests validate the functionality and correctness of the contract across individual functions and complete workflows.

---

## Overview of Auction Contract

The Auction Smart Contract provides the following features:

1. **Start Auction**: Allows the owner to start an auction.
2. **Place Bid**: Allows users to place bids during the auction.
3. **End Auction**: Enables the owner to end the auction and transfer funds.
4. **Get Highest Bid**: Retrieves the highest bid and bidder details.

---

## Testing Approach

The tests are divided into:

### Unit Tests
- Validate individual functions of the smart contract in isolation.
- Example scenarios:
  - Only the owner can start the auction.
  - Users can only place bids when the auction is active.
  - Bids lower than the highest bid are rejected.
  - Only the owner can end the auction.

### Integration Tests
- Validate interactions between multiple functions.
- Test the entire auction lifecycle:
  - Start the auction.
  - Place multiple bids.
  - End the auction and transfer funds.

---

## Prerequisites

Ensure you have the following installed:

- [Node.js](https://nodejs.org/)
- [Hardhat](https://hardhat.org/)

Install project dependencies:
```bash
npm install --save-dev hardhat chai ethers
```

---

## Test Implementation

### Unit Tests
File: `test/auction.unit.js`

Unit tests validate individual functions such as `startAuction`, `placeBid`, and `endAuction`. Example:

```javascript
it("should allow bids only when auction is active", async function () {
  await expect(auction.connect(bidder1).placeBid({ value: ethers.utils.parseEther("1") }))
    .to.be.revertedWith("Auction is not active");

  await auction.connect(owner).startAuction();
  await auction.connect(bidder1).placeBid({ value: ethers.utils.parseEther("1") });

  const highestBid = await auction.highestBid();
  expect(highestBid).to.equal(ethers.utils.parseEther("1"));
});
```

### Integration Tests
File: `test/auction.integration.js`

Integration tests validate the complete auction lifecycle:

```javascript
it("should correctly handle the full auction lifecycle", async function () {
  // Start Auction
  await auction.connect(owner).startAuction();
  expect(await auction.isActive()).to.be.true;

  // Place Bids
  await auction.connect(bidder1).placeBid({ value: ethers.utils.parseEther("1") });
  await auction.connect(bidder2).placeBid({ value: ethers.utils.parseEther("2") });

  const { highestBidder, highestBid } = await auction.getHighestBid();
  expect(highestBidder).to.equal(bidder2.address);
  expect(highestBid).to.equal(ethers.utils.parseEther("2"));

  // End Auction
  const ownerInitialBalance = await ethers.provider.getBalance(owner.address);

  await auction.connect(owner).endAuction();
  expect(await auction.isActive()).to.be.false;

  // Verify funds transfer to owner
  const ownerFinalBalance = await ethers.provider.getBalance(owner.address);
  expect(ownerFinalBalance).to.be.gt(ownerInitialBalance);
});
```

---

## Running the Tests

1. Run the following command to execute all tests:
```bash
npx hardhat test
```

2. View the test results in the console. Any failures will indicate discrepancies in the contract's behavior.

---

## Key Features Validated

1. **Ownership Restrictions**:
   - Only the owner can start and end the auction.

2. **Bid Validations**:
   - Bids are only accepted when the auction is active.
   - Lower bids are rejected.

3. **Funds Handling**:
   - Funds are correctly transferred to the owner upon ending the auction.

4. **Lifecycle Integrity**:
   - Complete workflows are tested for expected results.

---

## Additional Notes

- Ensure the smart contract is deployed to a local or test network before running the tests.
- Use the Hardhat Network for a quick and isolated test environment.

---

## License

This project is licensed under the MIT License.
