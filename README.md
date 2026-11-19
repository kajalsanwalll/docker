Docker
---

docker builds using layers to optimize size.   
it uses existing images(if any) from the local storage, i.e. caching otherwise it gets them from webcontainer.     

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


## Running an image

docker run <image_id> 
      or  
docker run -d <image_id>  //detach mode (run in the background)        
      or     
docker run --name my_container <image_id>  //giving specific container name   
or  
docker run -p 8080:80 <image_id>  //whatever on 80th container should be seen in my system at port 8080  
or  
docker run -v $(pwd):/app <image_id>  // load my current working directory onto the container  
or  
docker run -it <image_id> sh  //an interactive shell of the container opens in our terminal 




