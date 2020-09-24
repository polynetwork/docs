<h1 align="center">Ontology Cross Chain - Ethereum Relayer Manual</h1>
<h4 align="center">Version 1.0 </h4>

English | [中文](ethereum_relayer_manual_CN.md)

The relay chain serves as the intermediary for cross chain transactions both from Ethereum to another target chain and vice versa. The relayer that acts as the medium of communication between Ethereum and the relay chain synchronizes the block and transmits cross chain transactions from Ethereum to the relay chain and the other way around.

## Initial Setup

First, you need an Ethereum account with a certain amount of ETH balance, since sending transactions consumes gas. You also need to register your address on the relay in order to synchronize transaction headers and send cross chain transactions to the relay chain.

## Downloads

You need to download the Ethereum relayer package. Please follow [this]() link to download the executable binary

## Configuration

The following is a sample configuration of the Ethereum relayer:

```json
{
  "MultiChainConfig": {
    "RestURL": "http://172.168.3.73:40336",
    "EntranceContractAddress": "0300000000000000000000000000000000000000",
    "WalletFile": "./wallet.dat",
    "WalletPwd": "123456",
    "GasPrice": 0,
    "GasLimit": 2000000
  },
  "ETHConfig": {
    "RestURL": "https://ropsten.infura.io/v3/1ba5f3635395470e9a3f19ba7a852144",
    "LockerContractAddress": "0x3C3e443F4076aff3ded4C6a65217F9796BF3e6a7",
    "PrivateKey": "30E292FD40E645AB65FD3376689E6E5AC8C74E275867C4E7A04B88FA5CB4D252",
    "Signer": "0xB7Ee265D94446F465dba65002A9960D4bef9dca7",
    "AccountWalletPath": "./ethwallet.dat",
    "CapitalOwnersPath": "./capital-owners",
    "CapitalPassword": "123456"
  }
}
```

The configuration process consists of relay chain configuration and Ethereum configuration

Relay chain configuration components-

```
RestURL                          -           Node URL of the relay chain
EntranceContractAddress          -           Cross chain management contract of the relay chain
WalletFile&WalletPwd             -           Relay chain wallet account and password
GasPrice&GasLimit                -           Gas configuration of the relay chain
```

Ethereum configuration components:

```
RestURL                           -           Ethereum node URL
LockerContractAddress             -           Ethereum cross chain management contract
PrivateKey&Signer                 -           Ethereum account private key
```

## Execution

The two parameters used to run the relayer are as follows:

```
--ethereum             -    Used to specify the starting height of Ethereum chain, generally set to current height
--Poly             -    Used to specify the starting height of relay chain, generally set to 0, for non zero values the particular block needs to contain verification node information
```

The command to run the relayer is of the form:

```shell
./eth_relayer --ethereum 7318671 --Poly 0
```

The relayer can also be run in the background or run using the shell script (However, the starting parameters in the script would need to be modified)

```shell
./eth_relayer_start.sh
```

## License

The Poly Network library is licensed under the GNU Lesser General Public License v3.0. Please refer to the LICENSE file in the root directory of the project for details.
