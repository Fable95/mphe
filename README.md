# MPHE
Multi Party Homomorphic Encryption Library. This library builds on top of the HE library [SEAL](https://github.com/microsoft/SEAL/) and the MPC library [MP-SPDZ](https://github.com/data61/MP-SPDZ). As a baseline it enables the evaluation of all key generation and decryption algorithms of the SEAL framework in a multiparty fashion. For a more detailed discussion on how to install and setup MP-SPDZ and SEAL, we refer the reader to the respective repositories.

## Setting up the library
First create the `bin` folder used to store executables and key material for MPC:
```
mkdir bin/
```
As a baseline, the underlying frameworks need to be installed first.
```
git submodule update --init --recursive
```
As the submodules are loaded, we can patch our changes to MP-SPDZ into its directory:
```
cd MP-SPDZ
git apply ../mphe.patch
cd ..
```
For MP-SPDZ, the following dependencies are required:
```
sudo apt-get install automake build-essential clang cmake git libboost-dev libboost-filesystem-dev libboost-iostreams-dev libboost-thread-dev libgmp-dev libntl-dev libsodium-dev libssl-dev libtool python3
```
SEAL and MP-SPDZ work with different build systems, with cmake and Makefile respectively. To combine the two approaches, we recommend installing SEAL (which is easier to handle) into the MPHE folder, to simplify building the MPHE library. From the root folder execute:
```
cd SEAL
cmake -S . -B build -DCMAKE_INSTALL_PREFIX=<project-root>/MP-SPDZ/MPHE/seal -DSEAL_THROW_ON_TRANSPARENT_CIPHERTEXT=OFF  
cmake --build build
cmake --install build
cd ..
``` 
Note: The flag for transparent ciphertexts is necessary as intermediate steps in the Galois Key generation store incomplete ciphertext objects. 

Next, from the root directory install the dependencies of MP-SPDZ. This step requires an internet connection and may take a while as some dependencies are loaded from the web.
```
cd MP-SPDZ
make boost libote
```
Finally, we can compile our benchmarks with: 
```
make benchmark -j 8
```
This will move the executable into the `<project-root>/bin/` folder. Before we can execute it, we need to create the necessary key material. The fastest way to do this is:
```
cd bin/
bash ../MP-SPDZ/Scripts/setup-ssl.sh 2
touch HOSTS && printf "127.0.0.1\n127.0.0.1\n" > HOSTS
```
## Running the library
A concrete scenario of usage can be seen in `benchmark.h`, the executable of the benchmarks is in `benchmark-mphe-party.h`. It can be evaluated in one terminal with `./bench`. To run the protocol with more parties run it in multiple seperate terminal windows with 
```
./bench -p <num> -pn 8080 -ip HOSTS -N <num-parties>
```  
Note that with multiple parties, all ip addresses have to be stored in HOSTS, the port number has to be the same for all parties and the party number `<num>` must be ascending and starting from 0.

The compilation of benchmark will also create the static library `mphe.a` to integrate the api functions found in `mpheAPI.h`. The usage in `benchmark.h` showcases how the individual functions can be called. All MPC protocols that generate key material finally output SEAL compatible objects. In other words, keys generated with the MPHE library, can be serialized into a file and then later be used by a standalone seal application allowing for better interoperability. 

In the `mpheAPI.h`, we find the following shorthands:
- semiAPI
- mascotAPI
- shamirAPI
- malShamirAPI

These typedef classes provide shorthands for a selection of MPC schemes supported in MP-SPDZ. A particular application written for mpheAPI can be called with any of these instantiations and be directly evaluated in the respective MPC setting. Following the pattern in `benchmark.h`, we see the evaluation of all api functions can be done in the respective MPC settings by just calling the functions with different MPC parameters. Please note, that the shamir variants are only supported for n>3 parties in MP-SPDZ.
