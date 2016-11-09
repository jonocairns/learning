### common cloud application setup

I've only ever used Azure, so I want to add to this later but here's all I got on a basic web app setup.

For Azure, using the most specialised offerings tends to bring the cost down. If you have a simple frontend/backend setup I would use azure webapps. Webapps are almost container like and weed out a lot of the hard parts of web hosting (port forwarding / DNS etc).

I would split out the API and backend in to two completely seperate applications / pipelines. This means changes will be easier (if you're only changing the backend then you can just deploy the backend) and build steps can be chunked up and run in parallel (this is very important for later on when you scale out).

For the first cut, I would suggest have 2x website and 2x API. Then set up the instance scaling to a standard 'if CPU greater than X spawn new and continue'. Peering this enhanced monitoring and alerting is a must as it will require some tweaking in the initial phases. 
