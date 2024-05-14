=== ":simple-nodedotjs:{.icon .lg-icon .tab-icon} Node"

    To run this server application, you need [Node](https://nodejs.org/es/download/){:target="\_blank"} installed on your device.

    1. Navigate into the server directory
    ```bash
    cd openvidu-livekit-tutorials/application-server/node
    ```
    2. Install dependencies
    ```bash
    npm install
    ```
    3. Run the application
    ```bash
    node index.js
    ```

=== ":simple-goland:{.icon .lg-icon .tab-icon} Go"

    To run this server application, you need [Go](https://go.dev/doc/install){:target="\_blank"} installed on your device.

    1. Navigate into the server directory
    ```bash
    cd openvidu-livekit-tutorials/application-server/go
    ```
    2. Run the application
    ```bash
    go run main.go
    ```

=== ":simple-ruby:{.icon .lg-icon .tab-icon} Ruby"

    To run this server application, you need [Ruby](https://www.ruby-lang.org/en/downloads/){:target="\_blank"} installed on your device.

    1. Navigate into the server directory
    ```bash
    cd openvidu-livekit-tutorials/application-server/ruby
    ```
    2. Install dependencies
    ```bash
    bundle install
    ```
    3. Run the application
    ```bash
    ruby app.rb
    ```

=== ":fontawesome-brands-java:{.icon .lg-icon .tab-icon} Java"

    To run this server application, you need [Java](https://www.java.com/en/download/manual.jsp){:target="\_blank"} and [Maven](https://maven.apache.org){:target="\_blank"} installed on your device.

    1. Navigate into the server directory
    ```bash
    cd openvidu-livekit-tutorials/application-server/java
    ```
    2. Run the application
    ```bash
    mvn spring-boot:run
    ```

=== ":simple-python:{.icon .lg-icon .tab-icon} Python"

    To run this server application, you need [Python 3](https://www.python.org/downloads/){:target="\_blank"} installed on your device.

    1. Navigate into the server directory
    ```bash
    cd openvidu-livekit-tutorials/application-server/python
    ```
    2. Create a python3 environment and activate it
    ```bash
    python3 -m venv venv
    . venv/bin/activate
    ```
    3. Install dependencies
    ```bash
    pip install -r requirements.txt
    ```
    4. Run the application
    ```bash
    python3 app.py
    ```

=== ":simple-rust:{.icon .lg-icon .tab-icon} Rust"

    To run this server application, you need [Rust](https://www.rust-lang.org/tools/install){:target="\_blank"} installed on your device.

    1. Navigate into the server directory
    ```bash
    cd openvidu-livekit-tutorials/application-server/rust
    ```
    2. Run the application
    ```bash
    cargo run
    ```

=== ":simple-php:{.icon .lg-icon .tab-icon} PHP"

    To run this server application, you need [PHP](https://www.php.net/manual/en/install.php){:target="\_blank"} and [Composer](https://getcomposer.org/download/){:target="\_blank"} installed on your device.

    1. Navigate into the server directory
    ```bash
    cd openvidu-livekit-tutorials/application-server/php
    ```
    2. Install dependencies
    ```bash
    composer install
    ```
    3. Run the application
    ```bash
    composer start
    ```

=== ":simple-dotnet:{.icon .lg-icon .tab-icon} .NET"

    To run this server application, you need [.NET](https://dotnet.microsoft.com/en-us/download){:target="\_blank"} installed on your device.

    1. Navigate into the server directory
    ```bash
    cd openvidu-livekit-tutorials/application-server/dotnet
    ```
    2. Run the application
    ```bash
    dotnet run
    ```

    !!! warning

        This .NET server application needs the `LIVEKIT_API_SECRET` env variable to be at least 32 characters long. Make sure to udpate it [here](https://github.com/OpenVidu/openvidu-livekit-tutorials/blob/b97db7278227470fd386337ffde49b2458315c6f/application-server/dotnet/appsettings.json#L11){:target="\_blank"} and in your [LiveKit Server](#1-run-livekit-server).
