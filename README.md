# Library

## Lottie (animation)

#### Keyword:

- lottie, vue-lottie react-lottie, ngx-lottie

#### Lottie JSON

- Explore & download: [LottieFiles: Download Free](https://lottiefiles.com/)

# Flow

## Types of APIs:

1. `getSpins` - Get the remaining number of spins
2. `getTransactions` - Get the history of the most recent gifts
3. `getWheels` - Get the current lucky wheel (array of gifts)
4. `getPrizes` - Get the assets gained from the spin (how many airdrop points, mantles, ...)
5. `getProvider` - Returns the current provider, the api will return information such as prizes (what types of prizes are there)
6. `submitWheel` - Also known as spin api, sent to the server to get results when spinning

## Part 1: Init Screen

- Call api
  **getSpins** : Get the remaining number of spins
  **getWheels** : Get array of gifts
  **getProvider** : Get current provider
  **getPrizes** : Get prizes
  **getTransactions** :
  - Retrieve the closest winning result, then combine the _suffle_ **array of gift** obtained from the **_getWheels_** api to render the UI, so that the wheel display includes [0, 1, 2], ensuring the index of reward always 1.
  - Please see the `renderResult` function in the **SpinMachine.vue** component

## Part 2: Let's do it

The UI part of the spin includes 2 parts:

- 1 static lucky spin (display results)
- 1 animated lucky spin (shows scroll effect).

Based on the **isSpinning** variable, `display: none` will be applied to them. When **isSpinning** is true, the dynamic spin will be visible and the static spin will be hidden based on the `display: none` property and vice versa.

When the user clicks spin, the results **may not be returned immediately**. So we need a variable **isReceivingResult** to mark whether the **submitWheel** api is still loading or not?

Users **can spam** to start and stop continuously. So to ensure that the **number of times the user clicks the start** function is **equal** to the **number of clicks of the stop** function. So the stop function used a **until** utility to wait for the isReceivingResult result, then display the result & shoot fireworks.

### Start

1. Start the spin by setting `isSpinning` to true, at which point the dynamic spin will appear (Remember to check if there are still spins or not?)
2. Call submitWheel to get results from the server
3. Silently suffle the static spin when the result is available, so when stopped, the result will be immediately displayed in the middle of the 3 elements.
4. Set a 5-minute timeout to automatically stop

### Stop

1. **Must wait** for the results returned from the backend to stop. **Assuming the network is slow**, it takes up to 2 seconds to get the results, while the user can repeatedly click the stop button. The `until()` function has the effect of placing a promise to stop, which helps prevent spam. At the same time, if the user clicks start, then clicks stop (but the results are not yet available), it will **automatically** stop when the results become available. **This ensures that if a session has a start, it will also have a stop** (i.e., the number of calls to the start function is equal to the number of calls to the stop function).
2. Cleanup the `setTimeout` set in the `start()` function
3. Call the `fireAnimation()` function passed from the parent component
4. Call the `updateResult()` function to update the result passed in from the parent component. The purpose is to pass data from the child component to the parent component
