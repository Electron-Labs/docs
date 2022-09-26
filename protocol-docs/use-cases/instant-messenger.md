# Instant Messenger

Building a Privacy-Preserving Instant Messenger where sender, receiver, and message are completely hidden.

Consider users Alice and Bob who want to send messages to each other. Alice and Bob don’t know each other, say they are just random addresses on some blockchain. Say Alice has already sent a message to Bob by storing it as an encrypted state on-chain.

```json
//Decrypted value of prevMesg is a JSON as 
{
    sender : "alice_address",
    receiver : "bob_address",
    msg : "good morning"
}
```

Let us say now bob wants to reply to this message. Let’s see Bob can do that.

Step1: Read the encrypted state fro+m -n-chain

```cpp
type signature = {
    pubKey : int,
    message : string,
    sign : int,
}

type encrypted_json = {
    value: json,
    owner : signature,
}

program send_message {

    public inputs {
        encrypted_json prevMsg; //read this from the chain
        string input2;
    }

    private inputs {
        int input3;
        int input4;
        signature bob_sig;
        address address1;
    }

    //defining the output variables
    public outputs{
        encrypted output1;
        unencrypted output2;
    }
    
    //programme execution starts here
    main(){
        verify_ed25519(bob_sig.pubKey, bob_sig.message, bob_sig.sign);
        
        json pMsg = decrypt(prevMsg.value, bob_sig.pubKey);

        
        
        //your business logic goes here
        /*
```
