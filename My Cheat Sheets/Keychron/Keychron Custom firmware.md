# Keychron Custom firmware

To push the Bluetooth playground branch from the keychron people's fork of the qmk_firmware repo to your own Git repository, you can follow these general steps:

1. Fork the keychron people's qmk_firmware repository to create your own copy on GitHub. You can do this by clicking the "Fork" button on the qmk_firmware repository page on GitHub.
2. Clone your forked repository to your local machine using Git. You can do this by running the following command in your terminal:
    
    ```bash
    
    git clone https://github.com/your-username/qmk_firmware.git
    
    ```
    
    Be sure to replace "your-username" with your actual GitHub username.
    
3. Add the keychron people's qmk_firmware repository as a remote to your local Git repository. You can do this by running the following command:
    
    ```bash
    git remote add upstream https://github.com/keychron/qmk_firmware.git
    
    ```
    
4. Fetch the latest changes from the keychron people's repository by running the following command:
    
    ```bash
    git fetch upstream
    
    ```
    
5. Checkout the Bluetooth playground branch in your local repository by running the following command:
    
    ```bash
    git checkout -b bluetooth-playground upstream/bluetooth-playground
    
    ```
    
    This will create a new local branch called "bluetooth-playground" that is based on the keychron people's "bluetooth-playground" branch.
    
6. Make any changes you want to the code in your local repository.
7. Stage and commit your changes using Git. You can do this by running the following commands:
    
    ```bash
    git add .
    git commit -m "Your commit message here"
    
    ```
    
8. Push your changes to your forked repository on GitHub by running the following command:
    
    ```bash
    git push origin bluetooth-playground
    
    ```
    
    This will push your changes to a new branch called "bluetooth-playground" in your forked repository.
    
9. Finally, go to your forked repository on GitHub and create a pull request to merge your changes into the keychron people's "bluetooth-playground" branch. They will be able to review your changes and decide whether or not to accept the pull request.