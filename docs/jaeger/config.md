

![](../images/architecture-jaeger.png)
# Jaeger collector
- Agent which collects data and put to storage
```bash
 ./jaeger-collector --span-storage.type elasticsearch --es.server-urls http://42.118.242.146:9200 --collector.zipkin.http-port=9411
```
Port |	Protocol |	Function |
|-----|-----|----|
| 14267 | 	TChannel |	used by jaeger-agent to send spans in jaeger.thrift format |
|14250 |	gRPC |	used by jaeger-agent to send spans in model.proto format
|14268 |	HTTP |	can accept spans directly from clients in jaeger.thrift format over binary thrift protocol
|9411 |	HTTP |	can accept Zipkin spans in JSON or Thrift (disabled by default)
|14269 |	HTTP |	Healthcheck at / and metrics at /metrics

# Jaeger Query Service & UI
jaeger
```cmd
./jaeger-query --span-storage.type elasticsearch --es.server-urls http://42.118.242.146:9200
```
|Port |	Protocol |	Function |
|-----|-----|----|
|16686 |	HTTP 	| /api/* endpoints and Jaeger UI at /
| 16687 |	HTTP |	Healthcheck at / and metrics at /metrics

# Jaeger Agent
```cmd
./jaeger-agent --reporter.grpc.host-port=127.0.0.1:14250
```
Listener of jaeger client and push to jaeger-collector
|Port |	Protocol |	Function |
|-----|-----|----|
|6831 |	UDP |	accept jaeger.thrift in compact Thrift protocol used by most current Jaeger clients
|6832 |	UDP |	accept jaeger.thrift in binary Thrift protocol used by Node.js Jaeger client (because thriftrw npm package does not support compact protocol)
|5778 |	HTTP |	serve configs, sampling strategies
|5775 |	UDP |	accept zipkin.thrift in compact Thrift protocol (deprecated; only used by very old Jaeger clients, circa 2016)
|14271 |	HTTP |	Healthcheck at / and metrics at /metrics
