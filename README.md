Docker
---

docker builds using layers to optimize 
it uses existing images(if any) from the local storage, i.e. caching otherwise it gets them from webcontainer

RUN = used to execute commands at build time 
CMD = runs the commands at the container runtime.

After writing the basic dockerfile,
 docker build -t vite-test .

Now run the image,
--

but before running it, you need to fix vite.config.js to allow it to run as we didnt use nginx.

vite.config.js

server: {
    host : "0.0.0.0",
    port : 3000
},

run -p 3000:3000 vite-test


