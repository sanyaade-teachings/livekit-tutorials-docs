There are different options for running LiveKit Server:

=== "Run LiveKit locally"

	Follow the official instructions to [run LiveKit locally](https://docs.livekit.io/home/self-hosting/local/){target="_blank"}.

=== "Use LiveKit Cloud"

	You can free yourself from the burden of running LiveKit Server by using LiveKit Cloud directly. They offer a free layer that should cover the early stages of development: [LiveKit Cloud](https://cloud.livekit.io/){target="_blank"}.

=== "OpenVidu local deployment"

	OpenVidu is fully compatible with LiveKit. The OpenVidu local deployment has [a few advantages](../../../about-openvidu) over LiveKit when developing your application. You can easily run it on your local machine following these steps:

	1. Clone the repository
	```
	git clone https://github.com/OpenVidu/openvidu-local-deployment
	```

	2. Run a script to configure your local IP address in the deployment

		=== ":fontawesome-brands-windows:{.icon .lg-icon .tab-icon} Windows"

			```
			cd openvidu-local-deployment
			.\configure_lan_private_ip_windows.bat
			```

		=== ":simple-apple:{.icon .lg-icon .tab-icon} macOS"

			```
			cd openvidu-local-deployment
			./configure_lan_private_ip_macos.sh
			```

		=== ":simple-linux:{.icon .lg-icon .tab-icon} Linux"

			```
			cd openvidu-local-deployment
			./configure_lan_private_ip_linux.sh
			```

	3. Run the deployment
	```
	docker compose up
	```

=== "OpenVidu production deployment"

	If you want a production-ready deployment, visit the official [OpenVidu deployment guide](https://docs.openvidu.io/stable/installation/){target="_blank"}.