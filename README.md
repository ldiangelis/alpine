Create Token Project Notes:


1. This project details the creation of the "Alpine Life" Solana blockchain token in a containerized environment. As a quick disclaimer, this is not in any way financial advice, 
rather to work on a fun project.  The primary methods for the project include creating our own docker images, creating the crypto token itself, and finally deploying the 
token with associated metadata to show the logo and info about our token! Here is a link to the token on solscan.io - If interested in getting some free Alpine Life tokens 
hit me up and you got it! Although I should add the caveat that it is completely worthless.lol
Link: https://solscan.io/token/6uwWcAuNZ7i1PkU1BqUNJEmUNqgmXWE4Djq6eKMUuAAh

2. This tutorial was heavily inspired by NetworkChuck -> check out his excellent youtube channel here: https://www.youtube.com/@NetworkChuck

3. I personally like to run the container from terminal then attach Visual Studio Code (VSCode) -> this can be done by downloading the Docker extension.
Once installed we can run the container and attach VSCode to the container (Right click container from Docker tab, then select "Attach Visual Studio Code")
this is my preference for working with the container in a console setting

4. We are going to work with the terminal here in lieu of a dockerfile (or docker compose) as this is an excellent opportunity to build terminal skills.  As a bonus, it is 
fun to work with the terminal in general!  We implement this method as we want to be sure that all packages download correctly with the necessary dependencies.

5. We will discuss the token creation process for terminal, solana token-list commit, and why we have to use Strata Protocol to launch our token instead.

I: Pull ubuntu image and set up user for the container

1. Pull latest version of Ubuntu from docker hub:
	# sudo docker pull ubuntu:latest

2. Run ubuntu docker image

	# Get the image_id for our ubuntu image
		# sudo docker image ls
			# Copy and paste image_id to the run command below
			
	# Run the image
		# sudo docker run -p 8888:8888 -v /home/host_machine_user/host_machine_directory:/workspace/host_machine_directory -it [image_id] 

3. Add new user - this is an optional step, if you omit this you run as the root user

	# First run the following prereq's prior to adding the new user
		# apt update
			# this updates the package source list with the latest versions of the package repos
			
		# apt install nano
			# this is a simple text editor so we are able to read/write files as necessary
			
		# apt install sudo
			# downloads the sudo package so we require password to elevate priviledge for certain functions
			
		# passwd
			# set the sudo password
			
		# sudo groupadd docker
			# this creates a sudo group for docker
		
	# Add new user
		# sudo adduser [newuser]
			# here we define the new user name - can be whatever name you want
			# once user has been set, the terminal will prompt you to enter password for this user
			
		# sudo usermod -aG sudo [newuser]
			# this allows the user to have sudo prviledges
			
		# sudo usermod -aG docker [newuser]
			# this allows the user priviledges for our docker group
			
		# su [newuser]
			# quickly validate we are able to switch to our new user
			
		# sudo command
			# quickly run a command with our new user to verify we are able to run with sudo priviledges
			
		# su -
			# swap back to root user - if there is a problem with the [newuser], verify sudo/docker group permission
				# sudo -l -U [newuser]
					# for sudo priviledge
				# groups [newuser]
				
		# nano /etc/passwd
			# check to see what the User_ID and Group_ID are		
				# If you would like to modify the UID or GID use - both UID and GID are (int) dtypes:
					# usermod -u [newuserid] [newuser]
					# groupmod -g [newusergroupid] [newuser]
					
		# nano /etc/passwd
			# if you modified the UID and/or GID, verify they have been modified
		# If you modified [newuser] UID/GID - verify you still have sudo/docker group permissions
			# sudo -l -U [newuser]
			# groups [newuser]
			
		# If you have the correct permissions, stop the container so we can commit it:
			# sudo docker stop docker ps -q
			
		# Otherwise, follow the previous steps to re-add your newuser to sudo and docker groups
			# sudo usermod -aG sudo [newuser]
			# sudo usermod -aG docker [newuser]
			
	# Save the container on host machine and remove it
		# sudo docker container commit [container_id] [newimagename:tag]
		# sudo docker rm [container_id]
		
II: Install the prereq's for solana

	# Run the commited image we just created with our new user - if you skipped this above, omit the --user [newuser] flag
		# sudo docker run -p 8888:8888 --user [newuser] -v /home/host_machine_user/host_machine_directory:/home/host_machine_directory -it [imageid]
	
	# Install the necessary packages for solana
		# sudo apt install curl
			# this will allow us to download files/data from or to a server via https (others) - we are using https:
				# sh -c "$(curl -sSfL https://release.solana.com/v1.13.6/install)"
					# this installs the files we need to run our solana mainnet
					# verify this is the latest release 
						# https://github.com/solana-labs/solana/releases
							# version above is current as of Jan 17, 2023
					
		# exit the terminal and reopen it - see VSCode tip below as again, this is my preferred method, but not required
			# exit
				# exiting the terminal allows us to commit the changes to our /root/.profile file
				
		# VSCode tip:
			# this is where VSCode is really beneficial, we can keep the container running from host terminal,
	 		# and also have the benefit of being able to exit the VSCode terminal and reattach it to ensure our changes occur
	 			
		# sudo nano /root/.profile
			# this should automatically add the path from our curl above, but we should still verify it has been modified
			
		# verify the path updated
			# export PATH="/root/.local/share/solana/install/active_release/bin:$PATH"
			
		# cd ~/
			# change directory to [newuser] dir
			
		# curl https://sh.rustup.rs -sSF | sh
			# curl the https files and pipe it into the sh - sh here simply executes commands read from the curled file
		 	# Enter 1
			# exit terminal, reattach VSCode
			
		# cd ~/
			# change directory to [newuser] dir that contains .cargo extension
			
		# ls -a 
			# locate .cargo dir
				# sudo nano .cargo/env
					# Verify the path has been set to export PATH=$HOME/.cargo/bin:$PATH"
					# alternatively can run this without explicitly accessing this directory and file
						# source "$HOME/.cargo/env"
							
			
		# sudo apt install libudev-dev -y
		
		# sudo apt install libssl-dev pkg-config -y
		
		# sudo apt install build-essential -y
		
		# sudo cargo install spl-token-cli
			# note how we use cargo here, which is a rust command
			
		# stop the container
			# sudo docker stop docker ps -q
		
	# Commit the container and remove it
		# sudo docker container commit [container_id] [newimagename:tag]
		# sudo docker rm [container_id]
		
	# We now have ALL of the prereqs we need to create our token!

III. Create the token

	# Run the container - from last commit in Step II:	
		# sudo docker run -p 8888:8888 --user [newuser] -v /home/host_machine_user/host_machine_directory:/workspace/host_machine_directory -it [imageid]
	
	# Create the solana wallet -> this will create our solana wallet we use in the terminal (VSCode)
		# solana-keygen new
			Output:
				1. Public Wallet Address
				2. Seed Phrase
			# Remember to save the seed phrase for the wallet!! - This is how you will recover it for other wallets
			# Additionally, this will prompt you to add a BIP-39 Passphrase, while this is not required,
			it does offer more security - the only issue is that we are unable to use the seed phrase for
			a Solana wallet (such as Phantom wallet) - for now we should note that we can access the private 
			key here and then open it - this is useful so we can directly add the private key to our Phantom Wallet
			to get around the BIP-39 passphrase issue with importing the seed phrase to the wallet:
				# ~/.config/solana
				# nano json.id
				# We should note that we do not need to do this currently as we would not need this until we have completed all steps
				
		# Verify solana balance
			# solana balance
				# output will show SOL balance - this will be 0 until we fund our wallet in the next step
			# solana balance -v 
				# this will also display the path to the private key if you forget the path noted above
				
		# Send SOL to your new wallet from exchange of choice -- e.g. Kraken, Coinbase, etc.. (if funds on blockchain, use Solana compatible bridge to transfer funds) 
			# the sol wallet is the public address we want to send to 
			# we will need this going forward, so take the time now to add some SOL to your wallet - remember to add a decent bit of solana 
			# as we will have to use this to create our token, account, send tokens, etc. 
			# dont worry about adding too much, we can easily transfer our solana in the additional steps in section IV
			
		# Create our token
			# sudo spl-token create-token
				# Output:
					# [token address]: This is the public address for your token! We need this so make sure you have it copied somewhere.
					# so boom, we have now created our token!! Super easy!
					
		# Create our token account:
			# sudo spl-token create-account [token address]
				# We use the token address from our create-token method
			Output:
				# [token account] - this is the account we created that points to our token
				
		# Mint tokens:
			# spl-token mint [token address] [n_tokens] [token account]
			# syntax:
				# spl-token mint [token address] [n_tokens] [token account]
				# mint -> we want to mint our token
				# token address -> our token address
				# n_tokens -> number of tokens we want to mint - for example, 1000000000 (one-billion)
					# tip: use separators to keep track of value - much easier to read: e.g. 1_000_000_000
				# token account -> account we want to send our tokens to
				
		# Mint output:
			# The output will show the [token address] and the [token account] (where recipient is the token account address linked to our wallet)
				# the token is just one of the cryptos we hold in our wallet (account)
		
		# Verify the tokens have been minted to our account:
			# spl-token accounts
				# The output will show the number of tokens we have from our mint - this should equal the number of tokens we minted
			# spl-token accounts -v
				# this provides more info to include both the token address and the token account
		
		# Create a new phantom wallet (or use existing) so we can send some of our new tokens!
			# spl-token transfer --fund-recipient --allow-unfunded-recipient [token address] [n_tokens] [recipient wallet address]
			# syntax: spl-token transfer --fund-recipient [token address] [n_tokens] [recipient wallet address]
			# spl-token transfer -> initializing us to send our token
			# --fund-recipient -> saying we want to fund the recipient wallet address with our token (from token address)
			# --allow-unfunded-recipient -> we add this flag to indicate the recipient wallet does not currently have our token
			# token address -> our token
			# n_tokens -> number of tokens we want to send
			# recipient wallet address -> wallet we want to send tokens to
		
		# Access recipient wallet and verify tokens have been recieved
			# The tokens are now in our wallet, but there is no metadata associated with this token, we will add this shortly
		
		# Stop the container
			# sudo docker ps -a -q
		
		# Commit final image
			# sudo docker container commit [container id] [imagename:tag]
			# sudo docker rm [container_id]
		

IV: Work with final image - this contains all of our accounts, etc. and we can now pull up our container and get right to work when working with our token

		# Run our image:
			# sudo docker run -p 8888:8888 --user [newuser] -v /home/hostmachineuser/hostdir:/home/hostdir -it --rm [imageid]
				# here we finally implement the --rm flag as we have installed all the necessary packages for our token container
				# As such, the --rm flag automatically removes our container when we exit it - this drops the need to run the stop and remove commands once we exit the container
				
		# Work with our token:
			# Mint:
				We can also mint additional tokens should we want:
					spl-token mint [token address] [n_tokens] [account address]
				# However, if we would like to disable mint (as we should) we run:
			
			# Burn: - Lets say we minted an additional 1 billion tokens (2 billion in total), but we want to burn half of our supply
				# spl-token burn -v [account address] [n_tokens], where n_tokens = 1_000_000_000
			
			# Disable Mint:
				# spl-token authorize [token address] mint --disable
				
			# Transfer SPL-Token:
				# spl-token transfer --fund-recipient --allow-unfunded-recipient [token address] [n_tokens] [recipient wallet address]
				
			# Transfer Solana:
				# solana transfer [recipient wallet address] [n_tokens] --allow-unfunded-recipient
			# Help:
				# solana --help
				# spl-token --help
					# spl-token [option] --help - use this for more detailed info on specific option
			
		# Add json metadata to Solana Labs Token List:
			# First create a github repo for your token
				# Add logo for token (.png) file -> commit change
				# Click file and grab raw image path from download tab 
					# starts with "raw.githubusercontent.co/..."
			# Add the json list to the solana labs github token-list
			# Go to: github.com/solana-labs/token-list
				# Click fork
				# We have now forked to our github user
				# Go back to our user and hit the "." symbol -> this will launch VSCode in a webeditor 
				# Click assets -> click on the new folder button and paste the token address into the new folder
				# Right-click the folder and select upload -> select our .png image and upload
				# Next, select src -> tokens -> solana.tokenlist.json
					# Tip: Find a json file for existing token and copy the structure to add below
				# Scroll down to bottom (and add our token 1 BEFORE the last token) this helps maintain the proper strucutre..
				# Add our specific coin details [token address] [symbol] [name] [logo URI - raw github logo] and [tag]
				# Commit the change to our repo by clicking the "squiggly lines" (with the number over it)
				# Add message above your listed changes - should at least have token name to help identify if it sync's properly
					# e.g. Adding [token name] 
				# Create the pull request 
				# Wait for approval from Solana
				# Click pull requests at the top
					# If our token is listed we are good, otherwise the exception will indicate the issue
				# Check out solscan.io to see our token via the token address and if our metadata has sucessfull been added
				
		# So bummer...
			# Solana token list is no longer accepting pull requests.. so all of our work doesn't really matter..lol
			# Given this I went ahead and deleted my pull request, however here is the example of the json file we submit
			
{
      "chainId": 101,
      "address": "token address",
      "symbol": "ALPN",
      "name": "AlpineLife",
      "decimals": 9,
      "logoURI": "https://raw.githubusercontent.com/ldiangelis/alpine/main/logo.png",
      "tags": [
        "social-token"
    ]
} -- this token was not committed therefore the token exists, but is unrecognized on the network
			
		# To rectify this we can simply create a token on Strata Tokenlauch: https://app.strataprotocol.com/launchpad/create
			# We can add our logo, and metadata - Strata takes care of the rest
			# This is a basic process which is described well in their documentation - just a note, always add decimal places to your token, if you do not, this will be 
			# generated as an NFT!  
		
V. Summary

1. This was a fun project to create a docker container so we can create an environment to work with our crypto - this is really great because it allows us to have an isolated environment to work 
exclusively with our token

2. Also, working with the terminal is always fun!  However, we were unable to work directly in terminal and commit the changes to github - this is due to the fact that the
token-list has been "EOL" from solana labs github and therefore cannot add our metadata - I believe there are a few options out there - the only problem is that I disabled the mint function
and was unable to commit the metadata to other services.. That said, we went ahead and created our token and have sucessfully launched it via strata.. So we accomplished our goal, albeit outside
of the terminal!!
	
3. Something to note, our projects are only as good as the assumptions we make during implementation.  That in mind, my assumption that I could still commit to Solana token-list
was incorrect as I found that it was EOL (end of life) and therefore had to figure out another way that did not require terminal.. So always verify your project before beginning
to make sure you can complete all steps!  Oftentimes, the simplest aspect of the project is the one we overlook and therefore causes the biggest problems!

4. Also, just realizing now that I probably should have used Alpine Linux Distro.lol But what can I say, Ubuntu is awesome!
