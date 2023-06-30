# Reef Vue Template

This is a Vue template for building decentralized applications (dApps) on the Reef blockchain. It provides a basic structure and setup to get you started quickly. The template includes a demo application that you can run to see the functionalities in action.

## Installation

To use the template, follow these steps:

1. Clone this repository to your local machine.
2. Install the dependencies by running the following command:

```shell
yarn install
```

## Running the Demo App

To run the demo application, use the following command:

```shell
yarn serve
```

This will start the development server, and you can access the app in your browser at the specified URL.

## Building dApps

You can use this template as a starting point for building your own decentralized applications on top of the Reef blockchain. To do so, follow these steps:

1. Clone this repository to your local machine.
2. Replace the ABI (Application Binary Interface) of this project with the ABI of your smart contract. You can find the ABI in the JSON format provided by your smart contract's compiler.
3. Replace the address with the deployed address of your smart contract. Update the file located at `/src/utils/contract` with the new address.
4. Write your functions in the `App.jsx` file according to your dApp's requirements.

Please note that connections have already been established with the Reef WebSockets.

## Feedback and Contributions

If you have any feedback, questions, or suggestions, please feel free to open an issue in the GitHub repository. Contributions are also welcome!

## License

This project is licensed under the [MIT License](LICENSE).
