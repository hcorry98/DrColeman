# Dr. Brock Coleman

```
./deployWebsite.sh -k ~/Documents/Programming/DrColeman/colemanAWS.pem -h drbrockcoleman.com
```

## Steps to make a website:

### First, make a server instance:

1. Go to AWS account

2. Go to EC2

3. Select region (`US East (Ohio) - us-east-2` for the following server image)

4. Select `Launch Instance`

5. Name instance (Optional: use convention [owner]-[purpose]-[version] such as 260-webserver-base)

6. Search for the Amazon Machine Image (AMI ID: `ami-0b41d83057f814e3a`) - If you cannot find the AMI, make sure you are in the correct region as explained in Step 3

7. Select `Community AMIs` and select the image

8. Select t2.micro, t3.micro, or t3.nano for the instance type depending on resources needed and free options available

9. Create a new key pair and save somewhere *private and secure* (Optional: use convention [instanceName]-key)

10. Auto-assign public IP

11. Select or create security group (allow SSH, HTTP, and HTTPS from anywhere)

12. `Launch Instance`

13. Type public IP into web browser to test for success (`http://[public IP address]`)

### Second, assign an elastic (static) IP address:

1. Go to AWS account

2. Go to EC2

3. On the left, select `Network & Security | Elastic IPS`

4. Press the `Allocate Elastic IP address` button

5. Press the `Allocate` button

6. Select the newly displayed allocated address and edit it's name to give it a meaningful name

7. Press the `Actions` button

8. Selectt the `Associate Elastic IP address` option

9. Click on the `Instance` box and select the server instance you just created

10. Press Associate

### Third, purchase a domain name:

1. Go to AWS account

2. Go to Route 53

3. Select the `Domains > Registered domains` option from the menu on the left

4. Push the `Register Domain` option

5. Telect the TLD that you want. `.click` and `.link` are the cheapest. `click` does not offer privacy features, but `.link` does for just a little bit more money

6. Put the desired root domain into the search box and press the `Check` button. Search until you find an available root domain that you like

7. Press `Add to cart`

8. Fill out the contact details (this will be visible using the console program whois, unless privacy features were selected). If you use new contact information that a registry has never seen before it will require you to verify the email address. Follow the steps to verify your address.

9. Press `Continue`

10. Review everything and press `Complete Order`

### Fourth, assign the domain name and manage DNS records:

1. Go to AWS account

2. Go to Route 53

3. Select the `Hosted zones` option from the menu on the left

4. You should see your domain name listed here. If you don't, then the registration did not complete, or it is still pending. In that case go review the information found under `Domains > Pending request`

5. Click on your domain name to view the details. This will display existing DNS records with types such as `NS`, and `SOA`

6. First we will create our root domain DNS record. This will associate your domain name with your server's IP address and make it so you can use your domain name in the browser to navigate to your server

    i. Press the `Create record` button

    ii. In the value box enter the public IP address of your server (just the IP address, no HTTP or additional formatting)

    iii. Press `Create records`

    iv. A new `A` type record should appear in your list of records that represents the root domain name and your server's public IP address

7. Next we will create a DNS record that will map to your server for any subdomain of your root domain name. This is made possible because DNS allows you to specify wildcards for a DNS record.

    i. Press the `Create records` button

    ii. In the `Record name` box enter the text `*`. This wildcard represents that any subdomain, that is not explicitly defined by another DNS record, will match this record

    iii. In the `Value` box enter the public IP address of your server (again, just the IP address, no HTTP or additional formatting)

    iv. Press the `Create records` button

    v. A new `A` type record should appear in your list of records that represents the wildcard subdomain name and your server's public IP address

8. Test your domain name was assigned correctly by going to `http://[your domain]`

9. You may get a warning that your website is unsecure. We will handle this in the next section

10. Test any subdomain with `http://[any subdomain].[your domain]`

### Fifth, use Caddy to support HTTPS:

1. Open a console window

2. Use the `ssh` console program to shell into your production environment server

```
➜  ssh -i [key pair file] ubuntu@[yourdomainnamehere]
```

for example,

```
➜  ssh -i ~/keys/production.pem ubuntu@myfunkychickens.click
```

:warning: You will likely get an error stating that your key pair file permissions are too open. If so then you can restrict the permission on your file so that they are not accessible to all users by running the `chmod` console command:

```
`chmod 600 [key pair file]`
```

:warning: As it connects to the server it might warn you that it hasn't seen this server before. You can confidently say yes since you are sure of the identity of this server

Once you are connected, you are now looking at a console window for the web server that you launched and you should be in the ubuntu user's home directory. If you run `ls -l`, you should see the following:

```
➜  ls -l

total 4
lrwxrwxrwx 1 ubuntu ubuntu   20 Nov 17 23:03 Caddyfile -> /etc/caddy/Caddyfile
lrwxrwxrwx 1 ubuntu ubuntu   16 Nov 17 03:42 public_html -> /usr/share/caddy
drwxrwxr-x 6 ubuntu ubuntu 4096 Nov 30 22:42 services
```

The `Caddyfile` is the configuration file for your web service gateway. The `public_html` directory contains all of the static files that your are serving up directly through Caddy when using it as a web service. The `services` directory is the place where you are going to install all of your web services once you build them.

3. Edit Cadd's configuration file (`Caddyfile`) found in the ubuntu user's home directory. Note that since this file is owned by the root user, you need to use `sudo` to elevate your user to have the rights to change the file

```
➜  cd ~
➜  sudo vi Caddyfile
```

4. Modify the Caddy rule for handling requests to port 80 (HTTP), to instead handle request for your domain name. By not specifying a port, the rule will serve up files using port 443 (HTTPS), and any request to port 80 will automatically redirect the browser to port 443.

5. To edit the file using VI, press `i` and then you can type

6. Replace `:80` with your domain name (e.g. `myfunkychickens.click`). Make sure that you delete the colon

7. Delete the other two rules, as we are not concerned with using separate ports for certain subdomains

8. Review the Caddyfile to make sure it looks right. If your domain name was `myfunkychickens.click` it would look like the following:

```
myfunkychickens.click {
   root * /usr/share/caddy
   file_server
   header Cache-Control no-store
   header -etag
   header -server
}
```

9. Leave insertion mode by pressing `esc` and then save the file and exit VI by typing `:wq` and pressing enter

10. Restart Caddy so that your changes take effect

```
sudo service caddy restart
```

11. You make exit the remote shell by running the `exit` command

12. Use your browser to navigate to your domain name and you will see that the browser is displaying a lock icon, showing your website is using HTTPS

### Sixth, Uploading your HTML website:

1. Verify that your server is still running and that the default page is being displayed. If it is not, then you need to complete or review the previous steps

:warning: Do not continue until this works

3. [Fork this respository](https://github.com/webprogramming260/website-html) and clone it to your development environment. Using VS Code is recommended. For information about how to fork a GitHub repository read [this documentation](https://docs.github.com/en/get-started/quickstart/fork-a-repo). For information about cloning a repository to VS Code read [this](https://github.com/hcorry98/personalWebsite/blob/main/VSCode.md).

4. The repository contains four important files, `index.html`, `index.css`, `profile.png` and `deployWebsite.sh`. `index.html` contains a template HTML document. `deployWebsite.sh` contains a console shell script for deploying a new home page to your website. `index.css` and `profile.png` are files that will be utilized by `index.html` to enhance the look of the home page with styling and an image.

5. Modify `index.html` and `index.css` to how you want it. You can come back later to make it interactive with JavaScript

6. In a console window, change directory to your project directory, run the `deployWebsite.sh` script to push your changes to your deployment environment. This script takes two parameters, the PEM file to allow secure access to your server, and your server's domain name
```
./deployWebsite.sh  -k <yourpemkey> -h <yourdomain>
```
For example,
```
./deployWebsite.sh  -k ~/keys/production.pem -h funkychickens.click
```
:warning: Make sure you run the script in the project directory where the script resides

7. Open a browser window and verify that your new home page is showing up for your domain



