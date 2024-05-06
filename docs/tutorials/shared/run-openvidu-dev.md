Before you begin, make sure you have OpenVidu Server up and running. If you don't have it installed yet, you can easily set it up using our [Local Deployment](/installation/local/){target="_blank"}. Here's how:

=== ":fontawesome-brands-windows:{.icon .lg-icon .tab-icon} Windows"

	To run local deployment, you'll need [Docker](https://docs.docker.com/engine/){target="_blank"} installed on your development computer.

	1. Clone the repository
	```
	git clone https://github.com/OpenVidu/openvidu-local-deployment
	```
	2. Run a script to configure your local IP address in the deployment
	```
	cd openvidu-local-deployment
	.\configure_lan_private_ip_windows.bat
	```
	3. Run the deployment
	```
	docker compose up
	```

=== ":simple-apple:{.icon .lg-icon .tab-icon} macOS"

	To run local deployment, you'll need [Docker](https://docs.docker.com/engine/){target="_blank"} installed on your development computer.

	1. Clone the repository
	```
	git clone https://github.com/OpenVidu/openvidu-local-deployment
	```
	2. Run a script to configure your local IP address in the deployment
	```
	cd openvidu-local-deployment
	./configure_lan_private_ip_macos.sh
	```
	3. Run the deployment
	```
	docker compose up
    ```

=== ":simple-linux:{.icon .lg-icon .tab-icon} Linux"

	To run local deployment, you'll need [Docker](https://docs.docker.com/engine/){target="_blank"} installed on your development computer.

	1. Clone the repository
	```
	git clone https://github.com/OpenVidu/openvidu-local-deployment
	```
	2. Run a script to configure your local IP address in the deployment
	```
	cd openvidu-local-deployment
	./configure_lan_private_ip_linux.sh
	```
	3. Run the deployment
	```
	docker compose up
	```

!!! info "LiveKit compatible"

    OpenVidu is fully compatible with LiveKit Server, so you can also use the official [Livekit installation instructions](https://docs.livekit.io/realtime/self-hosting/server-setup/#running-locally){target="_blank"}.
