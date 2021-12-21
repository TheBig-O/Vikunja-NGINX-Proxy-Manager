# Vikunja-NGINX-Proxy-Manager

These are the basic instructions I used to set up Vikunja and proxy using NPM. 

When I installed [Vikunja](https://vikunja.io/) using Docker, I did so with the intention of proxying it with NGINX Proxy Manager (NPM). Unfortunately, there were no good examples of how to get everything working.  
While getting the basic setup working with an IP was painless, getting everything working with NPM was a nightmare. The frontend loaded properly through the web browser, but any attempt to login produced network errors while connecting to the API. As you'll recall, the API uses a different port than the frontend. So, when NPM tries to proxy to the API, it can't find it and an error is created.  
I looked on the internet and the best I could find was a set of instructions for straight NGINX. (I actually used the instructions for NGINX from the [Vikunja Docs](https://vikunja.io/docs/full-docker-example/#example-with-nginx-as-proxy). If you're familiar with the differences between NPM and NGINX, you'd be way ahead of me. I thought the NGINX instructions wouldn't help much since NPM uses a web interface in favor of traditional `.conf` files, but I was wrong.  You can directly edit the `.conf` file that pertains to the specific Proxy Host for Vikunja.  
After a bit of trial and error, I figured out how to get the API forwarded through the main Proxy Host.

>   Disclaimer: I don't claim that this is the best method. It's just the one that worked for me.

Here's what I did:
1. Create a standard Proxy Host for the Vikunja Frontend within NPM and point it to the URL you plan to use. The next several steps will enable the Proxy Host to successfully navigate to the API (on port 3456).
2. Verify that the page will pull up in your browser. (Do not bother trying to log in. It won't work. Trust me.)
3. Now, we'll work with the NPM container, so you need to identify the container name for your NPM installation. e.g. NGINX-PM
4. From the command line, enter `sudo docker exec -it [NGINX-PM container name] /bin/bash` and navigate to the proxy hosts folder where the `.conf` files are stashed. Probably `/data/nginx/proxy_host`. (This folder is a persistent folder created in the NPM container and mounted by NPM.)
5. Locate the `.conf` file where the server_name inside the file matches your Vikunja Proxy Host. Once found, add the following code, unchanged, just above the existing location block in that file. (They are listed by number, not name.)  
```
location ~* ^/(api|dav|\.well-known)/ {
        proxy_pass http://api:3456;
        client_max_body_size 20M;
    }
```
6. After saving the edited file, return to NPM's UI browser window and refresh the page to verify your Proxy Host for Vikunja is still online. 
7. Now, switch over to your Vikunja browswer window and hit refresh. If you configured your URL correctly in original Vikunja container, you should be all set and the browser will correctly show Vikunja. If not, you'll need to adjust the address in the top of the login subscreen to match your proxy address.


If you have questions, let me know and I'll try to help you solve them. I'm sure there are easier ways to do this, but this definitely worked for me.
