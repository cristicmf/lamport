# Hash based signatures
Quantum resistant lamport-diffie, winternitz and merkle tree hash based signature functions (in progress)..

Currently supports: LD-OTS, W-OTS and W-OTS+ keypair gen, signing and verification using sha256.
Additionally supports: MSS and XMSS(with W-OTS+) signing and verification for many-sign capability..XMSS is far faster but with larger keysizes

Todo: Add hypertree support for chained XMSS tree signatures..

<b>LD-OTS</b>

Lamport one time signature scheme creates 256 pairs of 256 bit private keys (in pairs at position 0 and 1) and a one way hash is computed to create an array of a further 256 pairs of 256 bit public keys. The entire 512x256 bit array of public keys is the public key. Signature involves creating a 256 bit hash of the message and then bitwise inspecting the hash result, for each bit - if 0 the first private key is the signature, if 1 the second is chosen as the signature. Obviously this is a one time signature. Public key size can be reduced to 256 bits by including 2,4,8 etc LD-OTS signatures in a merkle tree.

<b>Usage:</b>

To create lamport-diffie key pairs:

  <b>priv, pub = random_lkey()</b>
  
Returns lists of tuples containing 256 pairs of private keys and corresponding public keys.

To sign a message:

  <b>sig = sign_lkey(priv, message)</b>
  
Returns the signature which is a list of 256 256bit strings (50% of the private key).

To verify a signature:

  <b>verify_lkey(sig, message, pub)</b>
  
Returns true or false.
 
 
<b>Winternitz one time signature scheme (W-OTS)</b>

Based upon the initial scheme. Reduces key sizes and signatures significantly but increases computation time. Keypair generation creates 32 256 bit random private keys. When w=8 they are each hashed 255 times (2^w-1), then a further hash is performed, to create 32 256 bit public keys. Signature involves parsing the hash of the message 8 bits at a time (can be done 16 but this increases hash computation cycles significantly, from 0.01s to 3.7s), the 8 bit binary int (n) is subtracted from 256 and the private key is self hashed 256-n times. Thus the signature is 32 separate iterative hashes derived from the random private keys based up on the bitstream of the hash-message. To verify the message, the recipient knows the message and the public key and the signature, she simply parses the hash of the signature (8 bits at a time, extracting the 8 bit int, n) and then iteratively hashes the signature hashes n times. If the signature and message matches the public key then the public key is created from the signature. This is a one time signature which can be extended like the LD-OTS with a merkle tree to both reduce the public key size and increase the number of signs.

<b>Usage:</b> 

To create W-OTS key pairs:

  <b>priv, pub = random_wkey()</b>
  
Returns lists of 32 256 bit private keys and corresponding public keys.

To sign a message:

  <b>sig = sign_lkey(priv, message)</b>
  
Returns the signature which is a list of 32 256bit hashes.

To verify a signature:

<b>verify_lkey(sig, message, pub)</b>
  
Evaluates whether message signature is valid by recreating the public key from the signature. Returns true or false.




<b>Winternitz-OTS or LDOTS and Merkle Signature Scheme</b>

Creating more than one set of W-OTS keypairs and hashing the concatenated 32 256 public keys fragments for each keypair allows a merkle tree to be created. The pubhashes are concatenated and hashed upwards in a tree to the root. The same is true of the larger Lamport-Diffie OTS signatures. 

<b>Usage:</b>

To create a batch of W-OTS keypairs and link them to a merkle tree:

<b> data = random_wmss(20)</b>


To create n LD-OTS keypairs and link them to a merkle tree:

<b> data = random_ldmss(n) </b>

Returns confirmation of each n keypairs created and timings and details of the merkle tree. Data for each W-OTS/LD-OTS keypair are held in a list containing class objects with data accessible (e.g. data[0].pub (public key), data[0].priv (private key), data[0].concatpub (concatenated public key for pubhash), data[0].pubhash (hash of concatenated public key),data[0].type (type of key..LD or W-OTS), data[0].merkle_root (root hash of tree), data[0].merkle_path (hash authentication path from leaf to root). 

The merkle tree class is returned entirely in data[0].merkle_obj allowing similar access to associated data (e.g. 'data[0].merkle_obj.root') for the following: .root, .base, .height, .num_leaves, .num_branches .tree, .auth_lists. But everything needed to perform a signature with proofs to the merkle root is found in the data[n] class object.

Expect some computation time with larger merkle trees auth proofs. On a macbook pro: 4 signatures 0.003s, 32 s, 64 1.3s, 128 10.3s, 256 84.7s etc..

To sign a message using data from above two functions:

<b> sig = sign_mss(data, 'a message to sign', ots_key)</b>

Creates a signature using the private key from specified ots_key number within the data list. 

To verify a signature:

<b> verify_mss(sig, data, 'a message to sign', ots_key)</b>

Returns True or False.

To verify the merkle root can be computed using the public key and merkle path data:

<b> verify_root(pub, merkle_root, merkle_path)</b>

Returns True or False.

<b>Example code</b>

For example the following would create 64 W-OTS keypairs and build a merkle tree, print the merkle root, sign a message from keypair 32 (33rd), then verify that message and confirm the merkle authentication route is true.
<b>


  d = random_wmss(64)
  
  print d[0].merkle_root
  
  sig = sign_mss(d, 'this is probably quantum secure', 32)
  
  verify_mss(sig, d, 'this is probably quantum secure', 32)
  
  verify_root(d[32].pub, d[32].merkle_root, d[32].merkle_path)</b>
 

