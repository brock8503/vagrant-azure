# Vagrant Azure Provider

[![Gem Version](https://badge.fury.io/rb/vagrant-azure.png)](https://rubygems.org/gems/vagrant-azure)

This is a [Vagrant](http://www.vagrantup.com) 1.7.3+ plugin that adds [Microsoft Azure](https://azure.microsoft.com)
provider to Vagrant, allowing Vagrant to control and provision machines in Microsoft Azure.

## Usage

[Download Vagrant](http://www.vagrantup.com/downloads.html)

Install the vagrant-azure plugin using the standard Vagrant 1.1+ installation methods. After installing the plugin, you can ```vagrant up``` and use ```azure``` provider. For example:

```sh
& vagrant plugin install vagrant-azure --plugin-version '2.0.0.pre3'
...
$ vagrant up --provider=azure
...
```

You'll need an ```azure``` box before you can do ```vagrant up``` though.

## Quick Start

You can use the dummy box and specify all the required details manually in the ```config.vm.provider``` block in your ```Vagrantfile```. Add the dummy box with the name you want:

```sh
$ vagrant box add azure https://github.com/azure/vagrant-azure/raw/v2.0/dummy.box
...
```

For ***nix**, edit your `Vagrantfile` as shown below and provide all the values as explained.

```ruby
Vagrant.configure('2') do |config|
  config.vm.box = 'azure'

  # use local ssh key to connect to remote vagrant box
  config.ssh.private_key_path = '~/.ssh/id_rsa'
  config.vm.provider :azure do |azure, override|

    # use Azure Active Directory Application / Service Principal to connect to Azure
    # see: https://azure.microsoft.com/en-us/documentation/articles/resource-group-create-service-principal-portal/

    # each of the below values will default to use the env vars named as below if not specified explicitly
    azure.tenant_id = ENV['AZURE_TENANT_ID']
    azure.client_id = ENV['AZURE_CLIENT_ID']
    azure.client_secret = ENV['AZURE_CLIENT_SECRET']
    azure.subscription_id = ENV['AZURE_SUBSCRIPTION_ID']
  end

end
```

For **Windows**, edit your `Vagrantfile` as shown below and provide all the values as explained.

```ruby
Vagrant.configure('2') do |config|
  config.vm.box = 'azure'

  config.vm.provider :azure do |azure, override|

    # use Azure Active Directory Application / Service Principal to connect to Azure
    # see: https://azure.microsoft.com/en-us/documentation/articles/resource-group-create-service-principal-portal/

    # each of the below values will default to use the env vars named as below if not specified explicitly
    azure.tenant_id = ENV['AZURE_TENANT_ID']
    azure.client_id = ENV['AZURE_CLIENT_ID']
    azure.client_secret = ENV['AZURE_CLIENT_SECRET']
    azure.subscription_id = ENV['AZURE_SUBSCRIPTION_ID']

    azure.vm_image_urn = 'MicrosoftSQLServer:SQL2016-WS2012R2:Express:latest'
    azure.instance_ready_timeout = 600
    azure.vm_password = 'TopSecretPassw0rd'
    azure.admin_username = "OctoAdmin"
    azure.admin_password = 'TopSecretPassw0rd'
    override.winrm.transport = :ssl
    override.winrm.port = 5986
    override.winrm.ssl_peer_verification = false # must be false if using a self signed cert
  end

end
```

Now you can run

```sh
$ vagrant up --provider=azure
...
```

This will bring up an Azure VM as per the configuration options set above.

You can now either SSH (if its a *Nix VM) using ```vagrant ssh```, RDP (if its a Windows VM) using ```vagrant rdp``` or PowerShell ```vagrant powershell```.

Normally, a lot of this options, e.g., ```vm_image_urn```, will be embedded in a box file and you just have to provide minimal options in the ```Vagrantfile```. Since, we're using a dummy box, there are no pre-configured defaults.

## Azure Boxes

The vagrant-azure plugin provides the ability to use ```Azure``` boxes with Vagrant. Please see the example box provided in [example_box](https://github.com/azure/vagrant-azure/tree/v2.0/example_box) directory and follow the instructions there to build an ```azure``` box.

Please see [Vagrant Docs](http://docs.vagrantup.com/v2/) for more details.

## Configuration

The vagrant-azure provide exposes a few Azure specific configuration options:

### Mandatory

For instructions on how to setup an Azure Active Directory Application see: <https://azure.microsoft.com/en-us/documentation/articles/resource-group-create-service-principal-portal/>

* `tenant_id`: Your Azure Active Directory Tenant Id.
* `client_id`: Your Azure Active Directory application client id.
* `client_secret`: Your Azure Active Directory application client secret.
* `subscription_id`: The Azure subscription Id you'd like to use.

### Optional

* `resource_group_name`: (Optional) Name of the resource group to use.
* `location`: (Optional) Azure location to build the VM -- defaults to `westus`
* `vm_name`: (Optional) Name of the virtual machine
* `vm_password`: (Optional for *nix) Password for the VM -- This is not recommended for *nix deployments
* `vm_size`: (Optional) VM size to be used -- defaults to 'Standard_DS2_v2'. See sizes for [*nix](https://azure.microsoft.com/en-us/documentation/articles/virtual-machines-linux-sizes/), [Windows](https://azure.microsoft.com/en-us/documentation/articles/virtual-machines-windows-sizes/).
* `vm_image_urn`: (Optional) Name of the virtual machine image urn to use -- defaults to 'canonical:ubuntuserver:16.04-LTS:latest'. See documentation for [*nix](https://azure.microsoft.com/en-us/documentation/articles/virtual-machines-linux-cli-ps-findimage/), [Windows](https://docs.microsoft.com/en-us/azure/virtual-machines/virtual-machines-windows-cli-ps-findimage).
* `virtual_network_name`: (Optional) Name of the virtual network resource
* `subnet_name`: (Optional) Name of the virtual network subnet resource
* `instance_ready_timeout`: (Optional) The timeout to wait for an instance to become ready -- default 120 seconds.
* `instance_check_interval`: (Optional) The interval to wait for checking an instance's state -- default 2 seconds.
* `endpoint`: (Optional) The Azure Management API endpoint -- default `ENV['AZURE_MANAGEMENT_ENDPOINT']` if exists, falls back to <https://management.azure.com>.
* `admin_username`: (Optional) The root/administrator username for the VM
* `admin_password`: (Optional, Windows only) The password to set for the windows administrator user
* `winrm_install_self_signed_cert`: (Optional, Windows only) Whether to install a self-signed cert automatically to enable WinRM to communicate over HTTPS (5986). Only available when a custom `deployment_template` is not supplied. Default 'true'.
* `deployment_template`: (Optional) A custom ARM template to use instead of the default template
* `wait_for_destroy`: (Optional) Wait for all resources to be deleted prior to completing Vagrant destroy -- default false.
* `endpoint`: (Optional) The Azure Management API endpoint -- default 'https://management.azure.com' seconds -- ENV['AZURE_MANAGEMENT_ENDPOINT'].

### Notes
Using ARM it is still possible to use a custom VM if it has been hosted to a storage account.

* https://portal.azure.com/
* https://manage.windowsazure.com/

Make sure before starting we’ve run https://azure.microsoft.com/en-us/documentation/articles/resource-group-authenticate-service-principal-cli/

To host the vm I still need to figure out how to:
1.) Create a VM that https://github.com/Microsoft/azure-vhd-utils can upload to
- Create a resource group and storageaccount (general purpose)
  - TODO; grab automation option and see if it can integrate with azure cli
2.) Figure out how to convert a VDI to VHD in virtual box, what I know:
- Uninstall vbox tools
- Delete all snapshots
- Run `vboxmanage clonehd WinXP.vdi /winxp.vhd --format VHD`
- Launch VHD and sysprep generalize before
  - If sysprep fails to initialize, super hack https://blog.jamiebaldanza.org/2010/03/31/windows-7-sysprep-could-not-initialize-there-are-one-or-more-windows-updates-that-require-a-reboot/
  - Program and Features -> Turn Windows features on or off -> Remove or Add a single game -> restart VM
Note: Current the VHD is crashing in vbox when I load it
- Fixed: Vbox VM should be configured to use IDE controller vs SATA controller
Example:
azure-vhd-utils --verbose upload --localvhdpath ./packer-virtualbox-iso-1461003945-disk1_copy.vhd --stgaccountname mycustomvm2 --stgaccountkey UnoAzxfpIPyFyGI7ZcdrO6L2L7w80S4FhRtzhWpzhKK8kIgzOS6WcNfKNaJgZZFE+hjhGqQNskuca0LdAWm8jA== --blobname win7office365azure-vhd-utils --verbose upload --localvhdpath /Users/kylemcfarland/Development/vagrant-azure/packer-virtualbox-iso-1461003945-disk1_copy.vhd --stgaccountname mycustomvm2 --stgaccountkey UnoAzxfpIPyFyGI7ZcdrO6L2L7w80S4FhRtzhWpzhKK8kIgzOS6WcNfKNaJgZZFE+hjhGqQNskuca0LdAWm8jA== --blobname win7office365

3.) Complete the addition of the templat json in this project
- Reference: https://azure.microsoft.com/en-us/documentation/articles/virtual-machines-linux-cli-deploy-templates/#create-a-custom-vm-image

4.) Test the automation with `bundle exec vagrant`
Notes: nokigiri was causing problems required `gem install nokogiri -v 1.6.3.1 -- --use-system-libraries`
- gem install bundle -v 1.10.5
- bundle _1.10.5_
- bundle _1.10.5_ exec vagrant up --provider=azure --debug

TODO: Currently vagrant can’t connect to the vm through winrm, this is a requirement to check
vm status
https://docs.microsoft.com/en-us/azure/virtual-machines/virtual-machines-windows-winrm
https://github.com/Azure/azure-quickstart-templates/blob/master/201-vm-winrm-keyvault-windows/azuredeploy.json

# Getting Started

## Pre-requesites macOS
1.) Install Virtualbox with [brew](http://brew.sh/)

    # Must have cask to install virtualbox
    $ brew tap caskroom/cask
    $ brew cask install virtualbox

1.) Install vagrant

    $ brew cask install vagrant

3.) Install node

    $ brew install nvm
    $ echo “source $(brew --prefix nvm)/nvm.sh” >> .bash_profile
    $ nvm install node

## Pre-requesites ubunutu
1.) Install Virtualbox

    $ wget -q https://www.virtualbox.org/download/oracle_vbox_2016.asc -O- | sudo apt-key add -
    $ wget -q https://www.virtualbox.org/download/oracle_vbox.asc -O- | sudo apt-key add -i
    $ sudo apt-get update
    $ sudo apt-get install virtualbox-5.1

1.) Install vagrant

    $ sudo apt-get install vagrant

2.) Install node

    $ curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.0/install.sh | bash
    # Add to bash profile
    printf “export NVM_DIR='$HOME/.nvm’\n[ -s '$NVM_DIR/nvm.sh' ] && . $NVM_DIR/nvm.sh” >> .bash_profile

## Setup azure
1.) Create and log into [https://portal.azure.com](https://portal.azure.com) Windows Live account.
**Note** Make sure you have an active subscription https://account.windowsazure.com/Subscriptions
2.) Install azure cli

    $ npm install -g azure-cli

3.) Log into azure with account

    $ azure login
    # Make sure azure is in arm mode
    $ azure config mode arm

4.) Create the Active Directory and Service Principle. These are the primary parts for all resources created in azure.

    # Give your app a name
    $ azure ad sp create -n exampleapp > ad_sp.log

5.) Get the **AppId** and **ObjectId** from `ad_sp.log`.

    $ npm install -g azure_details
    $ azure_details -n <app_name>
    $ sed ‘/regex/g’ ad_sp.log
    TODO: Create node module to parse this out and remap names (stupid MS)
      azure account show <--- ID == subscription_id, Tenant_ID == tenant_id
      azure ad sp show -c <app_name> --json <-- objectId == object_id, appId == client_id

6.) Set your permissions to **owner** on the AD and SP

    $ azure role assignment create --objectId {object_id} -o Owner -c /subscriptions/{subscription_id}/

7.) Get the **client_key** from the portal.
    - Get your AD key url details `azure_details -n <app_name> --getADUrl`
        TODO generate https://portal.azure.com/#blade/Microsoft_AAD_IAM/ApplicationBlade/objectId/70dab975-7fbd-424d-8f90-01ca9c9a64c4/appId/d505af12-0aad-437d-af9b-083a29af4b13
    - Open the output from `getADUrl` in a browser
    - Create a new key under `All Settings -> Keys`
    - Copy and save the key for later use `azure_details -n <app_name> --client_key <key_info>`

8.) Create key ~/.ssh/azure_vagrant_rsa

    $ ssh-keygen -t rsa -b 4096 -C "your_email@email.com"

## Create reference VM
You will need a reference Windows VHD that is has been `sysprep` and generalized for distribution

1.) Refer to [https://github.com/joefitzgerald/packer-windows](https://github.com/joefitzgerald/packer-windows)
    TODO Create a windows vanilla vagrant box and add instruction on adding this box

2.) Generalize the image.

    # TODO This most likely does not work correctly
    $ vagrant ssh
    $ Sysprep /generalize /shutdown /oobe

**Note** Make sure there are no snapshots and if there are problems generalizing try this [hack](https://blog.jamiebaldanza.org/2010/03/31/windows-7-sysprep-could-not-initialize-there-are-one-or-more-windows-updates-that-require-a-reboot/)

3.) Create the VDH after it is generalized to upload to azure

    $ vboxmanage clonehd window_vm.vdi /output_to_upload.vhd --format VHD

4.) Create a storage account for the output vhd

    $ azure group create --name examplegroup --location westus --subscription {subscription_id}
    $ azure storage account create --kind Storage --location westus --resource-group examplegroup --subscription {subscription_id} --sku-name LRS examplestorage

5.) Get the **storage_key** from the portal.
    - Get your AD key url details `azure_details -n <app_name> --getStorageAccountUrl`
        TODO generate https://portal.azure.com/#resource/subscriptions/01f02320-1ffd-4b3d-9018-d2c91058f00a/resourceGroups/examplegroup/providers/Microsoft.Storage/storageAccounts/exmaplestorage/keys
    - Open the output from `getStorageAccountUrl` in a browser
    - Create a new key under `All Settings -> Keys`
    - Copy and save the key for later use `azure_details -n <app_name> --storage_name --storage_key <key_info>`

6.) Upload the vhd to azure

    $ azure-vhd-utils --verbose upload --localvhdpath ./output_to_upload.vhd --stgaccountname examplestorage --stgaccountkey UnoAzxfpIPyFyGI7ZcdrO6L2L7w80S4FhRtzhWpzhKK8kIgzOS6WcNfKNaJgZZFE+hjhGqQNskuca0LdAWm8jA== --blobname windows_vm_template

## Create a Azure VM from template
Now that all the image template is uploaded an Azure VM can be created through the `vagrant-azure` plugin.

1.) Install the plugin

    vagrant plugin install vagrant-azure --plugin-version 'some_version' --plugin-prerelease
