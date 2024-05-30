# Elastic stack (ELK) on Docker

Run the latest version of the [Elastic stack][elk-stack] with Docker and Docker Compose.

It gives you the ability to analyze any data set by using the searching/aggregation capabilities of Elasticsearch and
the visualization power of Kibana.

Based on the [official Docker images][elastic-docker] from Elastic:

- [Elasticsearch](https://github.com/elastic/elasticsearch/tree/main/distribution/docker)
- [Logstash](https://github.com/elastic/logstash/tree/main/docker)
- [Kibana](https://github.com/elastic/kibana/tree/main/src/dev/build/tasks/os_packages/docker_generator)

Other available stack variants:

- [`tls`](https://github.com/deviantony/docker-elk/tree/tls): TLS encryption enabled in Elasticsearch, Kibana (opt in),
  and Fleet
- [`searchguard`](https://github.com/deviantony/docker-elk/tree/searchguard): Search Guard support

## tl;dr

```sh
docker-compose up setup
```

```sh
docker-compose up
```

![Animated demo](https://user-images.githubusercontent.com/3299086/155972072-0c89d6db-707a-47a1-818b-5f976565f95a.gif)

---

# Ý tưởng

Dùng đơn giản để ghi và quản lý log của ứng dụng Spring boot bằng công cụ mạnh mẽ ELK

Cấu hình mặc định của dự án này được cố tình tối giản và không có ý kiến. Nó
không phụ thuộc vào bất kỳ sự phụ thuộc bên ngoài nào và sử dụng ít tính năng tự động hóa tùy chỉnh nếu cần thiết để thiết lập mọi thứ và
đang chạy.
Đơn giản hơn của [Repo Origin](https://github.com/deviantony/docker-elk)
Ko dùng filebeat mà từ ứng dụng Spring boot gửi các log qua tcp port: 50000 đến logstach
Từ logstach fillter thành các @metadata rồi chuyển đến elasticsearch
Từ elaticsearch lưu dữ liệu sau đó chuyển sang Kibana để hiển thị lên giao diện

## Đây là cấu hình của mình trong Spring boot

Phải cài thêm thư viện mình sủe dụng build.gradle nên thêm dependencies này

```
	implementation 'net.logstash.logback:logstash-logback-encoder:6.6'
```

Trong file lockback-spring.xml

```
<configuration>

    <appender name="LOGSTASH" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
        <destination>localhost:50000</destination>
        <encoder class="net.logstash.logback.encoder.LogstashEncoder" >
        </encoder>
    </appender>
    <root level="INFO">
        <appender-ref ref="LOGSTASH" />
    </root>

</configuration>
```

Đơn giản vậy là khi bạn gọi log bằng các thư viện log thì log sẽ tự động được gửi qua logback.

### Cấu hình file logstash.conf

Đây là file cấu hình quan trọng của logstash ở đây thì cấu hình input là tcp port:50000
Filter theo level log (INFO, WARN, ERROR, ...) rồi gửi vào logstash

### Host setup

- [Docker Engine][docker-install] version **18.06.0** or newer
- [Docker Compose][compose-install] version **1.28.0** or newer (including [Compose V2][compose-v2])
- 1.5 GB of RAM

> [!NOTE]
> Especially on Linux, make sure your user has the [required permissions][linux-postinstall] to interact with the Docker
> daemon.

By default, the stack exposes the following ports:

- 50000: Logstash TCP input
- 9600: Logstash monitoring API
- 9200: Elasticsearch HTTP
- 9300: Elasticsearch TCP transport
- 5601: Kibana

> [!WARNING]
> Elasticsearch's [bootstrap checks][bootstrap-checks] were purposely disabled to facilitate the setup of the Elastic
> stack in development environments. For production setups, we recommend users to set up their host according to the
> instructions from the Elasticsearch documentation: [Important System Configuration][es-sys-config].

### Docker Desktop

#### Windows

If you are using the legacy Hyper-V mode of _Docker Desktop for Windows_, ensure [File Sharing][win-filesharing] is
enabled for the `C:` drive.

#### macOS

The default configuration of _Docker Desktop for Mac_ allows mounting files from `/Users/`, `/Volume/`, `/private/`,
`/tmp` and `/var/folders` exclusively. Make sure the repository is cloned in one of those locations or follow the
instructions from the [documentation][mac-filesharing] to add more locations.

## Usage

> [!WARNING]
> You must rebuild the stack images with `docker-compose build` whenever you switch branch or update the
> [version](#version-selection) of an already existing stack.

### Bringing up the stack

Clone this repository onto the Docker host that will run the stack with the command below:

```sh
git clone https://github.com/deviantony/docker-elk.git
```

Then, initialize the Elasticsearch users and groups required by docker-elk by executing the command:

```sh
docker-compose up setup
```

If everything went well and the setup completed without error, start the other stack components:

```sh
docker-compose up
```

> [!NOTE]
> You can also run all services in the background (detached mode) by appending the `-d` flag to the above command.

Give Kibana about a minute to initialize, then access the Kibana web UI by opening <http://localhost:5601> in a web
browser and use the following (default) credentials to log in:

### Initial setup

#### Setting up user authentication

Để đơn giản hơn thì phần này đã loại bỏ, có thể bật nó trong elasticsearch.yml

#### Injecting data

Launch the Kibana web UI by opening <http://localhost:5601> in a web browser, and use the following credentials to log
in:

Đơn giản nhất tôi truyền data qua tcp 50000.
Now that the stack is fully configured, you can go ahead and inject some log entries.

Bạn có thể tải dữ liệu mẫu mà Kibana cung cấp

### Cleanup

Theo mặc định, dữ liệu Elaticsearch được lưu giữ bên trong một ổ đĩa.

Để tắt hoàn toàn ngăn xếp và xóa tất cả dữ liệu đã tồn tại, hãy sử dụng lệnh Docker Compose sau:

````sh
docker-compose down -v

### Version selection

Phiên bản được sử dụng là 8.13 bạn có thể chuyển đổi phiên bản trong file .env

Đây là thông báo quan trọng nếu bạn muốn chuyển đổi phiên bản
> [!IMPORTANT]
> Always pay attention to the [official upgrade instructions][upgrade] for each individual component before performing a
> stack upgrade.

Older major versions are also supported on separate branches:

- [`release-7.x`](https://github.com/deviantony/docker-elk/tree/release-7.x): 7.x series
- [`release-6.x`](https://github.com/deviantony/docker-elk/tree/release-6.x): 6.x series (End-of-life)
- [`release-5.x`](https://github.com/deviantony/docker-elk/tree/release-5.x): 5.x series (End-of-life)

## Configuration

> [!IMPORTANT]
> Configuration is not dynamically reloaded, you will need to restart individual components after any configuration
> change.

### Nếu bạn muốn cấu hình lại elasticsearch

The Elasticsearch configuration is stored in [`elasticsearch/config/elasticsearch.yml`][config-es].

You can also specify the options you want to override by setting environment variables inside the Compose file:

```yml
elasticsearch:
  environment:
    network.host: _non_loopback_
    cluster.name: my-cluster
````

Please refer to the following documentation page for more details about how to configure Elasticsearch inside Docker
containers: [Install Elasticsearch with Docker][es-docker].

### Nếu bạn muốn cấu hình lại Kibana

The Kibana default configuration is stored in [`kibana/config/kibana.yml`][config-kbn].

You can also specify the options you want to override by setting environment variables inside the Compose file:

```yml
kibana:
  environment:
    SERVER_NAME: kibana.example.org
```

Please refer to the following documentation page for more details about how to configure Kibana inside Docker
containers: [Install Kibana with Docker][kbn-docker].

### Cấu hình lại logstash

The Logstash configuration is stored in [`logstash/config/logstash.yml`][config-ls].

You can also specify the options you want to override by setting environment variables inside the Compose file:

```yml
logstash:
  environment:
    LOG_LEVEL: debug
```

Please refer to the following documentation page for more details about how to configure Logstash inside Docker
containers: [Configuring Logstash for Docker][ls-docker].

### How to disable paid features

Mình đã sử dụng Basic License. TThu viện gốc sử dụng Trial License.

### How to scale out the Elasticsearch cluster

Follow the instructions from the Wiki: [Scaling out Elasticsearch](https://github.com/deviantony/docker-elk/wiki/Elasticsearch-cluster)

### Để thực hiện lại các cấu hình các kiểu :))

To run the setup container again and re-initialize all users for which a password was defined inside the `.env` file,
simply "up" the `setup` Compose service again:

```console
$ docker-compose up setup
 ⠿ Container docker-elk-elasticsearch-1  Running
 ⠿ Container docker-elk-setup-1          Created
Attaching to docker-elk-setup-1
...
docker-elk-setup-1  | [+] User 'monitoring_internal'
docker-elk-setup-1  |    ⠿ User does not exist, creating
docker-elk-setup-1  | [+] User 'beats_system'
docker-elk-setup-1  |    ⠿ User exists, setting password
docker-elk-setup-1 exited with code 0
```

## Extensibility

Bạn có thể thêm các Plugin
Đã xóa file extentions trong repo cha. bao gồm cấu hình filebeat hướng dẫn của nó và các exxtentions khác.

### Plugins and integrations

See the following Wiki pages:

- [External applications](https://github.com/deviantony/docker-elk/wiki/External-applications)
- [Popular integrations](https://github.com/deviantony/docker-elk/wiki/Popular-integrations)
