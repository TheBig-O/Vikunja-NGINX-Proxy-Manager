# Vikunja-NGINX-Proxy-Manager

These are the basic instructions I used to set up Vikunja and proxy using NPM. 

When I installed [Vikunja](https://vikunja.io/) using Docker, I did so with the intention of proxying it with NGINX Proxy Manager (NPM). Unfortunately, there were no good examples of how to get everything working.
While getting the basic setup working with an IP was painless, getting everything working with NPM was a nightmare. The frontend loaded properly through the web browser, but any attempt to login produced network errors while connecting to the API. As you'll recall, the API uses a different port than the frontend. So, when NPM tries to proxy to the API, it can't find it and an error is created.
I looked on the internet and the best I could find was a set of instructions for straight NGINX. (I actually used the instructions for NGINX from the [Vikunja Docs](https://vikunja.io/docs/full-docker-example/#example-with-nginx-as-proxy). If you're familiar with the differences between NPM and NGINX, you'd be way ahead of me. I thought the NGINX instructions wouldn't help much since NPM uses a web interface in favor of traditional `.conf` files, but I was wrong.  You can directly edit the `.conf` file that pertains to the specific Proxy Host for Vikunja.
After a bit of trial and error, I figured out how to get the API forwarded through the main Proxy Host.

>   Disclaimer: I don't claim that this is the best method. It's just the one that worked for me.

Here's what I did:
1. Set up the Vikunja Docker stack/container.
2. Verify that you can log in and use Vikunja through the IP used for setup.
3. Create your Proxy Host within NPM and point it to whatever URL you plan to use and verify that the page will pull up in your browser. Do not bother trying to log in. It won't work. (Trust me.)
4. Next, you need to edit some of the NPM files directly. For this, you'll need to know the container name for your NPM installation. I used NGINX-PM, as you see below. From the command line, enter `sudo docker exec -it NGINX-PM /bin/bash`.
5. Navigate to the proxy hosts folder where the `.conf` files are stashed. In my case, the directory is: `/data/nginx/proxy_host`. Inside this folder, you'll see files for each of your Proxy Hosts. (They're numbered, not named. So, you'll have to `cat` each of them to find the correct file to edit. What you're looking for is a `server_name` that matches your Vikunja Proxy Host.)
Once found, edit the file to add the following. (You don't need to change anything in the following)
```
location ~* ^/(api|dav|\.well-known)/ {
        proxy_pass http://api:3456;
        client_max_body_size 20M;
    }
```
I added these lines just above the other `location` block and ensured that the spacing and indents matched.
6. After saving the edited file, return to NPM's browser window and refresh the page. Ensure your Proxy Host for Vikunja is still online.
7. The last step is to return to the Vikunja web page and adjust the address in the top of the login subscreen. Initially it will say "Sign in to your Vikunja account on" and your IPAddress:3456. It needs to be changed to `[YourDomanain]/api/v1`. Upon clicking the "Change" button, you'll be greeted by a green banner telling you the connection was successful.
8. And finally, you can log in and start using Vikunja.

If you have questions, let me know and I'll try to help you solve them. I'm sure there are easier ways to do this, but this definitely worked for me.
