# How to create an interactive Solidity lab with Hardhat?

We'll divide this part into 5 sections:

1. Creating lab metadata
2. Setting up lab defaults
3. Setting up lab challenges
4. Setting up evaluation script
5. Setting up test file

## Introduction

This guide would assume that you already have created an interactive course from your instructor panel. If not, [go here and set it up first](https://codedamn.com/instructor/interactive-courses)

## Step 1 - Creating lab metadata

<!--@include: ./../_components/LabMetadata.md-->

### Lab Details

Lab details is the tab where you add two important things:

-   Lab title
-   Lab description

This can be used as a helper text area for your lab. You should include as much detail as possible here to make it easier for the user to understand and solve your coding lab.

Let's move to the next tab now.

### Container Image

Container image should be set as `Web 3.0` for Solidity labs.

![Setting a container image](/images/solidity/image-selection.png)

We'll be cloning and using a playground that sets up a Hardhat project from scratch.

### Lab Layout

<!--@include: ./../_components/LabLayout.md-->

## Step 2 - Lab Defaults

Lab defaults section include how your lab environment boots. It is one of the most important parts because a wrong default environment might confuse your students. Therefore it is important to set it up properly.

When a codedamn playground boots, it can setup a filesystem for user by default. You can specify what the starting files could be, by specifying a git repository and a branch name.

:::tip
For Solidity, we recommend you to fork the following repository and use it as a starter template: [Solidity + Hardhat template - codedamn](https://github.com/codedamn-classrooms/web3-controller)
:::

:::info
You will find a `.cdmrc` file in the repository given to you above. It is highly recommend, at this point, that you go through the [.cdmrc guide and how to use .cdmrc in playgrounds](/docs/concepts/cdmrc) to understand what `.cdmrc` file exactly is. Once you understand how to work with `.cdmrc` come back to this area.
:::

Let us briefly understand what this repository does.

1. It uses hardhat as the toolkit which can be customized from the `hardhat.config.js` file.
2. Inside of `preview` folder, there is a simple `server.js` file written. This file, alongside `ui.jsx` and `doc.html` provides a very simple UI for the user to manage the contracts.
3. These contracts are written inside `contracts` folder. They are automatically complied by the `server.js` whenever it changes.
4. The `.cdmrc` file starts this `preview.js` server and allows user to write smart contracts in-browser while compiling them automatically.

:::info
You can choose to remove the `preview` folder completely, and customize `.cdmrc` file according to your needs. You can read more about `.cdmrc` [and how to use it here](/docs/concepts/cdmrc).
:::

## Step 3 - Lab challenges

<!--@include: ./../_components/LabChallenges.md-->

## Step 4 - Evaluation Script

Evaluation script is the first code that actually runs when the user on the codedamn playground clicks on `Run Tests` button.

![](/images/common/lab-run-tests.png)

Remember that we're using the hardhat utility in this lab to test.

Therefore, your testing script would look like as follows:

```sh
# move the test file to user directory (hardhat testing util requires it)
mkdir -p $USER_CODE_DIR/test
mv $TEST_FILE_NAME $USER_CODE_DIR/test

# run hardhat testing util assuming we have correct mocha settings
cd $USER_CODE_DIR
yarn --silent hardhat test > $UNIT_TEST_OUTPUT_FILE

# run a light node script to extract out results and write them back in expected format

cat > /home/damner/.test/process-results.js << EOF
const fs = require('fs')
const fileData = fs.readFileSync(process.env.UNIT_TEST_OUTPUT_FILE, { encoding: 'utf8' })
const payload = JSON.parse(fileData.slice(fileData.indexOf('{')))
const answers = payload?.tests?.map((result, i) => {
    if(result.err.stack) {
        console.error(result.err.message)
        return false
    } else {
        console.log(\`Test ${i} passed\`)
        return true
    }
}) || []
fs.writeFileSync(process.env.UNIT_TEST_OUTPUT_FILE, JSON.stringify(answers))
EOF

node /home/damner/.test/process-results.js
```

You might need to have a little understanding of bash scripting. Let us understand how the evaluation bash script is working:

1. We initially create a `test` directory inside of the user folder. This is because we'll then run `yarn hardhat test` which requires the tests to be placed inside of the `test` directory.
2. We move `$TEST_FILE_NAME` inside this directory. `$TEST_FILE_NAME` is a special filename. It is the complete path of a file which is downloaded when the user clicks on `Run Test` button. This file is editable (in the next step).
3. We then run `yarn --silent hardhat test` and then output it's result into a file contained in env variable `$UNIT_TEST_OUTPUT_FILE`. The value inside this variable maps to a special file that the playground **eventually** reads to determine the status of all the challenges.
4. We then create a very simple script in JS that reads the output by `hardhat test`. Note that this output is generated by `mocha`. It is important that this output is in JSON. You **have** to make sure that your `hardhat.config.js` is as follows:

```js
// ...other code
module.exports = {
	// ...other code
	solidity: '0.8.4',
	mocha: {
		// * the reporter here needs to be JSON
		reporter: 'json',
	},
}
```

4. We then convert the JSON result dump into a boolean array and write it back on the same `$UNIT_TEST_OUTPUT_FILE`
5. This file is then read by the playground, and a corresponding `PASS` or `FAIL` is assigned to every challenge. For example, if we finally wrote `[true, false, false, true]`, this would mean that Challenge 1 passed, Challenge 2 and 3 failed, and Challenge 4 passed.

:::note
At this point we're yet to write the testing script that is executed by `yarn hardhat test`. We'll do that in the next step now.
:::

**Note:** You can setup a full testing environment in this block of evaluation script (installing more packages, etc. if you want). However, your bash script test file will be timed out **after 30 seconds**. Therefore, make sure, all of your testing can happen within 30 seconds.

## Step 5 - Test file

You will see a button named `Edit Test File` in the `Evaluation` tab. Click on it.

![](/images/common/lab-edit-test.png)

When you click on it, a new window will open. This is a test file area.

You can write anything here. Whatever script you write here, can be executed from the `Test command to run section` inside the evaluation tab we were in earlier.

The point of having a file like this to provide you with a place where you can write your evaluation script.

:::note
For Solidity labs, I'm assuming you will be using the `hardhat test` alongside mocha to test the user code. Hence, we use the mocha testing syntax here.
:::

![](/images/solidity/template-pick.png)

Select `Solidity (Hardhat)`, and the following code should appear in your editor:

```js
const { expect } = require('chai')

describe('Token contract', function () {
	it('Deployment should assign the total supply of tokens to the owner', async function () {
		const [owner] = await ethers.getSigners()

		const Token = await ethers.getContractFactory('Token')

		const hardhatToken = await Token.deploy()

		const ownerBalance = await hardhatToken.balanceOf(owner.address)
		expect(await hardhatToken.totalSupply()).to.equal(ownerBalance)
	})
})
```

This test suite is written how a typical hardhat test suite should be written. You can find how to write hardhat test suites in their own documentation here: [Testing smart contracts with hardhat](https://hardhat.org/tutorial/testing-contracts#5.-testing-contracts)

Note:

-   There **must** be only 1 `describe` block here (this is an assumption in our evaluation script).
-   The number of `it(...)` functions inside your test file suite should match the number of challenges added in the instructor Evaluation tab area.
-   If your number of `it(...)` blocks are less than challenges added back in the UI, the "extra" UI challenges would automatically stay as "false". If you add more challenges in test file, the results would be ignored.

This completes your evaluation script for the lab. Your lab is now almost ready for users.

## Setup Verified Solution (Recommended)

<!--@include: ./../_components/LabVerifiedSolution.md-->
