# NFT-based-Event-Memorabilia
使用NFT为活动创建独特的数字纪念品，如音乐会门票或体育赛事门票。
from web3 import Web3
from solcx import compile_standard, install_solc
import json

# Replace YOUR_INFURA_PROJECT_ID with your actual Infura project ID
infura_url = "https://rinkeby.infura.io/v3/YOUR_INFURA_PROJECT_ID"
web3 = Web3(Web3.HTTPProvider(infura_url))

# Check if connected to blockchain
print("Connected to blockchain:", web3.isConnected())

# Compiling the Solidity code
install_solc('0.8.0')

# Solidity source code
solidity_src = """
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract EventMemorabilia is ERC721URIStorage, Ownable {
    constructor() ERC721("EventMemorabilia", "EVENT") {}

    function mintMemorabilia(address recipient, uint256 tokenId, string memory tokenURI) public onlyOwner {
        _mint(recipient, tokenId);
        _setTokenURI(tokenId, tokenURI);
    }
}
"""

compiled_sol = compile_standard({
    "language": "Solidity",
    "sources": {"EventMemorabilia.sol": {"content": solidity_src}},
    "settings": {
        "outputSelection": {
            "*": {
                "*": ["abi", "metadata", "evm.bytecode", "evm.bytecode.sourceMap"]
            }
        }
    },
},
solc_version='0.8.0',
)

# Deploying the contract
bytecode = compiled_sol['contracts']['EventMemorabilia.sol']['EventMemorabilia']['evm']['bytecode']['object']
abi = json.loads(compiled_sol['contracts']['EventMemorabilia.sol']['EventMemorabilia']['metadata'])['output']['abi']

# Set up your Ethereum account
# Replace 'your_private_key_here' with your Ethereum account private key
account = web3.eth.account.privateKeyToAccount('your_private_key_here')
chain_id = 4  # Rinkeby testnet ID

# Create the contract in Python
EventMemorabilia = web3.eth.contract(abi=abi, bytecode=bytecode)

# Build transaction
transaction = EventMemorabilia.constructor().buildTransaction({
    'from': account.address,
    'nonce': web3.eth.getTransactionCount(account.address),
    'gas': 1728712,
    'gasPrice': web3.toWei('50', 'gwei'),
    'chainId': chain_id
})

# Sign the transaction
signed_txn = account.sign_transaction(transaction)

# Send the transaction
tx_hash = web3.eth.sendRawTransaction(signed_txn.rawTransaction)

# Wait for the transaction to be mined
tx_receipt = web3.eth.wait_for_transaction_receipt(tx_hash)

# Contract deployment is successful
print("Contract deployed at:", tx_receipt.contractAddress)

# Minting an NFT for an event
# Create a function to mint NFT with event details
def mint_event_nft(event_name, event_date, ticket_id, owner_address):
    event_memorabilia = web3.eth.contract(address=tx_receipt.contractAddress, abi=abi)
    token_uri = f"https://myevent.com/nft/{event_name.replace(' ', '')}_{event_date}_{ticket_id}.json"  # A URL to the NFT metadata
    tx = event_memorabilia.functions.mintMemorabilia(owner_address, ticket_id, token_uri).buildTransaction({
        'from': account.address,
        'nonce': web3.eth.getTransactionCount(account.address),
        'gas': 1728712,
        'gasPrice': web3.toWei('21', 'gwei'),
        'chainId': chain_id
    })
    signed_tx = account.sign_transaction(tx)
    tx_hash = web3.eth.sendRawTransaction(signed_tx.rawTransaction)
    print(f"NFT minted for event {
