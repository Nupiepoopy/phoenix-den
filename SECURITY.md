# Security Policy

## Supported Versions

Use this section to tell people about which versions of your project are
currently being supported with security updates.

| Version | Supported          |
| ------- | ------------------ |
| 5.1.x   | :white_check_mark: |
| 5.0.x   | :x:                |
| 4.0.x   | :white_check_mark: |
| < 4.0   | :x:                |

## Reporting a Vulnerability

Use this section to tell people how to report a vulnerability.

Tell them where to go, how often they can expect to get an update on a
reported vulnerability, what to expect if the vulnerability is accepted or
declined, etc.

syntax = "proto3";

package envoy.extensions.filters.network.http_connection_manager.v3;

import "envoy/config/accesslog/v3/accesslog.proto";

import "envoy/config/core/v3/base.proto";

import "envoy/config/core/v3/config_source.proto";

import "envoy/config/core/v3/extension.proto";

import "envoy/config/core/v3/protocol.proto";

import "envoy/config/core/v3/substitution_format_string.proto";

import "envoy/config/route/v3/route.proto";

import "envoy/config/route/v3/scoped_route.proto";

import "envoy/config/trace/v3/http_tracer.proto";

import "envoy/type/tracing/v3/custom_tag.proto";

import "envoy/type/v3/percent.proto";

import "google/protobuf/any.proto";

import "google/protobuf/duration.proto";

import "google/protobuf/struct.proto";

import "google/protobuf/wrappers.proto";

import "envoy/annotations/deprecation.proto";

import "udpa/annotations/migrate.proto";

import "udpa/annotations/security.proto";

import "udpa/annotations/status.proto";

import "udpa/annotations/versioning.proto";

import "validate/validate.proto";

option java_package = "io.envoyproxy.envoy.extensions.filters.network.http_connection_manager.v3";

option java_outer_classname = "HttpConnectionManagerProto";

option java_multiple_files = true;

option (udpa.annotations.file_status).package_version_status = ACTIVE;

// [#protodoc-title: HTTP connection manager]

// HTTP connection manager :ref:`configuration overview <config_http_conn_man>`.

// [#extension: envoy.filters.network.http_connection_manager]

// [#next-free-field: 43]

message HttpConnectionManager {

  option (udpa.annotations.versioning).previous_message_type =

      "envoy.config.filter.network.http_connection_manager.v2.HttpConnectionManager";

  enum CodecType {

    // For every new connection, the connection manager will determine which

    // codec to use. This mode supports both ALPN for TLS listeners as well as

    // protocol inference for plaintext listeners. If ALPN data is available, it

    // is preferred, otherwise protocol inference is used. In almost all cases,

    // this is the right option to choose for this setting.

    AUTO = 0;

    // The connection manager will assume that the client is speaking HTTP/1.1.

    HTTP1 = 1;

    // The connection manager will assume that the client is speaking HTTP/2

    // (Envoy does not require HTTP/2 to take place over TLS or to use ALPN.

    // Prior knowledge is allowed).

    HTTP2 = 2;

    // [#not-implemented-hide:] QUIC implementation is not production ready yet. Use this enum with

    // caution to prevent accidental execution of QUIC code. I.e. `!= HTTP2` is no longer sufficient

    // to distinguish HTTP1 and HTTP2 traffic.

    HTTP3 = 3;

  }

  enum ServerHeaderTransformation {

    // Overwrite any Server header with the contents of server_name.

    OVERWRITE = 0;

    // If no Server header is present, append Server server_name

    // If a Server header is present, pass it through.

    APPEND_IF_ABSENT = 1;

    // Pass through the value of the server header, and do not append a header

    // if none is present.

    PASS_THROUGH = 2;

  }

  // How to handle the :ref:`config_http_conn_man_headers_x-forwarded-client-cert` (XFCC) HTTP

  // header.

  enum ForwardClientCertDetails {

    // Do not send the XFCC header to the next hop. This is the default value.

    SANITIZE = 0;

    // When the client connection is mTLS (Mutual TLS), forward the XFCC header

    // in the request.

    FORWARD_ONLY = 1;

    // When the client connection is mTLS, append the client certificate

    // information to the requestâ€™s XFCC header and forward it.

    APPEND_FORWARD = 2;

    // When the client connection is mTLS, reset the XFCC header with the client

    // certificate information and send it to the next hop.

    SANITIZE_SET = 3;

    // Always forward the XFCC header in the request, regardless of whether the

    // client connection is mTLS.

    ALWAYS_FORWARD_ONLY = 4;

  }

  // [#next-free-field: 10]

  message Tracing {

    option (udpa.annotations.versioning).previous_message_type =

        "envoy.config.filter.network.http_connection_manager.v2.HttpConnectionManager.Tracing";

    enum OperationName {

      // The HTTP listener is used for ingress/incoming requests.

      INGRESS = 0;

      // The HTTP listener is used for egress/outgoing requests.

      EGRESS = 1;

    }

    reserved 1, 2;

    reserved "operation_name", "request_headers_for_tags";

    // Target percentage of requests managed by this HTTP connection manager that will be force

    // traced if the :ref:`x-client-trace-id <config_http_conn_man_headers_x-client-trace-id>`

    // header is set. This field is a direct analog for the runtime variable

    // 'tracing.client_sampling' in the :ref:`HTTP Connection Manager

    // <config_http_conn_man_runtime>`.

    // Default: 100%

    type.v3.Percent client_sampling = 3;

    // Target percentage of requests managed by this HTTP connection manager that will be randomly

    // selected for trace generation, if not requested by the client or not forced. This field is

    // a direct analog for the runtime variable 'tracing.random_sampling' in the

    // :ref:`HTTP Connection Manager <config_http_conn_man_runtime>`.

    // Default: 100%

    type.v3.Percent random_sampling = 4;

    // Target percentage of requests managed by this HTTP connection manager that will be traced

    // after all other sampling checks have been applied (client-directed, force tracing, random

    // sampling). This field functions as an upper limit on the total configured sampling rate. For

    // instance, setting client_sampling to 100% but overall_sampling to 1% will result in only 1%

    // of client requests with the appropriate headers to be force traced. This field is a direct

    // analog for the runtime variable 'tracing.global_enabled' in the

    // :ref:`HTTP Connection Manager <config_http_conn_man_runtime>`.

    // Default: 100%

    type.v3.Percent overall_sampling = 5;

    // Whether to annotate spans with additional data. If true, spans will include logs for stream

    // events.

    bool verbose = 6;

    // Maximum length of the request path to extract and include in the HttpUrl tag. Used to

    // truncate lengthy request paths to meet the needs of a tracing backend.

    // Default: 256

    google.protobuf.UInt32Value max_path_tag_length = 7;

    // A list of custom tags with unique tag name to create tags for the active span.

    repeated type.tracing.v3.CustomTag custom_tags = 8;

    // Configuration for an external tracing provider.

    // If not specified, no tracing will be performed.

    //

    // .. attention::

    //   Please be aware that *envoy.tracers.opencensus* provider can only be configured once

    //   in Envoy lifetime.

    //   Any attempts to reconfigure it or to use different configurations for different HCM filters

    //   will be rejected.

    //   Such a constraint is inherent to OpenCensus itself. It cannot be overcome without changes

    //   on OpenCensus side.

    config.trace.v3.Tracing.Http provider = 9;

  }

  message InternalAddressConfig {

    option (udpa.annotations.versioning).previous_message_type =

        "envoy.config.filter.network.http_connection_manager.v2.HttpConnectionManager."

        "InternalAddressConfig";

    // Whether unix socket addresses should be considered internal.

    bool unix_sockets = 1;

  }

  // [#next-free-field: 7]

  message SetCurrentClientCertDetails {

    option (udpa.annotations.versioning).previous_message_type =

        "envoy.config.filter.network.http_connection_manager.v2.HttpConnectionManager."

        "SetCurrentClientCertDetails";

    reserved 2;

    // Whether to forward the subject of the client cert. Defaults to false.

    google.protobuf.BoolValue subject = 1;

    // Whether to forward the entire client cert in URL encoded PEM format. This will appear in the

    // XFCC header comma separated from other values with the value Cert="PEM".

    // Defaults to false.

    bool cert = 3;

    // Whether to forward the entire client cert chain (including the leaf cert) in URL encoded PEM

    // format. This will appear in the XFCC header comma separated from other values with the value

    // Chain="PEM".

    // Defaults to false.

    bool chain = 6;

    // Whether to forward the DNS type Subject Alternative Names of the client cert.

    // Defaults to false.

    bool dns = 4;

    // Whether to forward the URI type Subject Alternative Name of the client cert. Defaults to

    // false.

    bool uri = 5;

  }

  // The configuration for HTTP upgrades.

  // For each upgrade type desired, an UpgradeConfig must be added.

  //

  // .. warning::

  //

  //    The current implementation of upgrade headers does not handle

  //    multi-valued upgrade headers. Support for multi-valued headers may be

  //    added in the future if needed.

  //

  // .. warning::

  //    The current implementation of upgrade headers does not work with HTTP/2

  //    upstreams.

  message UpgradeConfig {

    option (udpa.annotations.versioning).previous_message_type =

        "envoy.config.filter.network.http_connection_manager.v2.HttpConnectionManager."

        "UpgradeConfig";

    // The case-insensitive name of this upgrade, e.g. "websocket".

    // For each upgrade type present in upgrade_configs, requests with

    // Upgrade: [upgrade_type]

    // will be proxied upstream.

    string upgrade_type = 1;

    // If present, this represents the filter chain which will be created for

    // this type of upgrade. If no filters are present, the filter chain for

    // HTTP connections will be used for this upgrade type.

    repeated HttpFilter filters = 2;

    // Determines if upgrades are enabled or disabled by default. Defaults to true.

    // This can be overridden on a per-route basis with :ref:`cluster

    // <envoy_api_field_config.route.v3.RouteAction.upgrade_configs>` as documented in the

    // :ref:`upgrade documentation <arch_overview_upgrades>`.

    google.protobuf.BoolValue enabled = 3;

  }

  reserved 27, 11;

  reserved "idle_timeout";

  // Supplies the type of codec that the connection manager should use.

  CodecType codec_type = 1 [(validate.rules).enum = {defined_only: true}];

  // The human readable prefix to use when emitting statistics for the

  // connection manager. See the :ref:`statistics documentation <config_http_conn_man_stats>` for

  // more information.

  string stat_prefix = 2 [(validate.rules).string = {min_len: 1}];

  oneof route_specifier {

    option (validate.required) = true;

    // The connection managerâ€™s route table will be dynamically loaded via the RDS API.

    Rds rds = 3;

    // The route table for the connection manager is static and is specified in this property.

    config.route.v3.RouteConfiguration route_config = 4;

    // A route table will be dynamically assigned to each request based on request attributes

    // (e.g., the value of a header). The "routing scopes" (i.e., route tables) and "scope keys" are

    // specified in this message.

    ScopedRoutes scoped_routes = 31;

  }

  // A list of individual HTTP filters that make up the filter chain for

  // requests made to the connection manager. :ref:`Order matters <arch_overview_http_filters_ordering>`

  // as the filters are processed sequentially as request events happen.

  repeated HttpFilter http_filters = 5;

  // Whether the connection manager manipulates the :ref:`config_http_conn_man_headers_user-agent`

  // and :ref:`config_http_conn_man_headers_downstream-service-cluster` headers. See the linked

  // documentation for more information. Defaults to false.

  google.protobuf.BoolValue add_user_agent = 6;

  // Presence of the object defines whether the connection manager

  // emits :ref:`tracing <arch_overview_tracing>` data to the :ref:`configured tracing provider

  // <envoy_api_msg_config.trace.v3.Tracing>`.

  Tracing tracing = 7;

  // Additional settings for HTTP requests handled by the connection manager. These will be

  // applicable to both HTTP1 and HTTP2 requests.

  config.core.v3.HttpProtocolOptions common_http_protocol_options = 35

      [(udpa.annotations.security).configure_for_untrusted_downstream = true];

  // Additional HTTP/1 settings that are passed to the HTTP/1 codec.

  config.core.v3.Http1ProtocolOptions http_protocol_options = 8;

  // Additional HTTP/2 settings that are passed directly to the HTTP/2 codec.

  config.core.v3.Http2ProtocolOptions http2_protocol_options = 9

      [(udpa.annotations.security).configure_for_untrusted_downstream = true];

  // An optional override that the connection manager will write to the server

  // header in responses. If not set, the default is *envoy*.

  string server_name = 10

      [(validate.rules).string = {well_known_regex: HTTP_HEADER_VALUE strict: false}];

  // Defines the action to be applied to the Server header on the response path.

  // By default, Envoy will overwrite the header with the value specified in

  // server_name.

  ServerHeaderTransformation server_header_transformation = 34

      [(validate.rules).enum = {defined_only: true}];

  // The maximum request headers size for incoming connections.

  // If unconfigured, the default max request headers allowed is 60 KiB.

  // Requests that exceed this limit will receive a 431 response.

  // The max configurable limit is 96 KiB, based on current implementation

  // constraints.

  google.protobuf.UInt32Value max_request_headers_kb = 29

      [(validate.rules).uint32 = {lte: 96 gt: 0}];

  // The stream idle timeout for connections managed by the connection manager.

  // If not specified, this defaults to 5 minutes. The default value was selected

  // so as not to interfere with any smaller configured timeouts that may have

  // existed in configurations prior to the introduction of this feature, while

  // introducing robustness to TCP connections that terminate without a FIN.

  //

  // This idle timeout applies to new streams and is overridable by the

  // :ref:`route-level idle_timeout

  // <envoy_api_field_config.route.v3.RouteAction.idle_timeout>`. Even on a stream in

  // which the override applies, prior to receipt of the initial request

  // headers, the :ref:`stream_idle_timeout

  // <envoy_api_field_extensions.filters.network.http_connection_manager.v3.HttpConnectionManager.stream_idle_timeout>`

  // applies. Each time an encode/decode event for headers or data is processed

  // for the stream, the timer will be reset. If the timeout fires, the stream

  // is terminated with a 408 Request Timeout error code if no upstream response

  // header has been received, otherwise a stream reset occurs.

  //

  // This timeout also specifies the amount of time that Envoy will wait for the peer to open enough

  // window to write any remaining stream data once the entirety of stream data (local end stream is

  // true) has been buffered pending available window. In other words, this timeout defends against

  // a peer that does not release enough window to completely write the stream, even though all

  // data has been proxied within available flow control windows. If the timeout is hit in this

  // case, the :ref:`tx_flush_timeout <config_http_conn_man_stats_per_codec>` counter will be

  // incremented. Note that :ref:`max_stream_duration

  // <envoy_api_field_config.core.v3.HttpProtocolOptions.max_stream_duration>` does not apply to

  // this corner case.

  //

  // If the :ref:`overload action <config_overload_manager_overload_actions>` "envoy.overload_actions.reduce_timeouts"

  // is configured, this timeout is scaled according to the value for

  // :ref:`HTTP_DOWNSTREAM_STREAM_IDLE <envoy_api_enum_value_config.overload.v3.ScaleTimersOverloadActionConfig.TimerType.HTTP_DOWNSTREAM_STREAM_IDLE>`.

  //

  // Note that it is possible to idle timeout even if the wire traffic for a stream is non-idle, due

  // to the granularity of events presented to the connection manager. For example, while receiving

  // very large request headers, it may be the case that there is traffic regularly arriving on the

  // wire while the connection manage is only able to observe the end-of-headers event, hence the

  // stream may still idle timeout.

  //

  // A value of 0 will completely disable the connection manager stream idle

  // timeout, although per-route idle timeout overrides will continue to apply.

  google.protobuf.Duration stream_idle_timeout = 24

      [(udpa.annotations.security).configure_for_untrusted_downstream = true];

  // The amount of time that Envoy will wait for the entire request to be received.

  // The timer is activated when the request is initiated, and is disarmed when the last byte of the

  // request is sent upstream (i.e. all decoding filters have processed the request), OR when the

  // response is initiated. If not specified or set to 0, this timeout is disabled.

  google.protobuf.Duration request_timeout = 28

      [(udpa.annotations.security).configure_for_untrusted_downstream = true];

  // The amount of time that Envoy will wait for the request headers to be received. The timer is

  // activated when the first byte of the headers is received, and is disarmed when the last byte of

  // the headers has been received. If not specified or set to 0, this timeout is disabled.

  google.protobuf.Duration request_headers_timeout = 41 [

    (validate.rules).duration = {gte {}},

    (udpa.annotations.security).configure_for_untrusted_downstream = true

  ];

  // The time that Envoy will wait between sending an HTTP/2 â€œshutdown

  // notificationâ€ (GOAWAY frame with max stream ID) and a final GOAWAY frame.

  // This is used so that Envoy provides a grace period for new streams that

  // race with the final GOAWAY frame. During this grace period, Envoy will

  // continue to accept new streams. After the grace period, a final GOAWAY

  // frame is sent and Envoy will start refusing new streams. Draining occurs

  // both when a connection hits the idle timeout or during general server

  // draining. The default grace period is 5000 milliseconds (5 seconds) if this

  // option is not specified.

  google.protobuf.Duration drain_timeout = 12;

  // The delayed close timeout is for downstream connections managed by the HTTP connection manager.

  // It is defined as a grace period after connection close processing has been locally initiated

  // during which Envoy will wait for the peer to close (i.e., a TCP FIN/RST is received by Envoy

  // from the downstream connection) prior to Envoy closing the socket associated with that

  // connection.

  // NOTE: This timeout is enforced even when the socket associated with the downstream connection

  // is pending a flush of the write buffer. However, any progress made writing data to the socket

  // will restart the timer associated with this timeout. This means that the total grace period for

  // a socket in this state will be

  // <total_time_waiting_for_write_buffer_flushes>+<delayed_close_timeout>.

  //

  // Delaying Envoy's connection close and giving the peer the opportunity to initiate the close

  // sequence mitigates a race condition that exists when downstream clients do not drain/process

  // data in a connection's receive buffer after a remote close has been detected via a socket

  // write(). This race leads to such clients failing to process the response code sent by Envoy,

  // which could result in erroneous downstream processing.

  //

  // If the timeout triggers, Envoy will close the connection's socket.

  //

  // The default timeout is 1000 ms if this option is not specified.

  //

  // .. NOTE::

  //    To be useful in avoiding the race condition described above, this timeout must be set

  //    to *at least* <max round trip time expected between clients and Envoy>+<100ms to account for

  //    a reasonable "worst" case processing time for a full iteration of Envoy's event loop>.

  //

  // .. WARNING::

  //    A value of 0 will completely disable delayed close processing. When disabled, the downstream

  //    connection's socket will be closed immediately after the write flush is completed or will

  //    never close if the write flush does not complete.

  google.protobuf.Duration delayed_close_timeout = 26;

  // Configuration for :ref:`HTTP access logs <arch_overview_access_logs>`

  // emitted by the connection manager.

  repeated config.accesslog.v3.AccessLog access_log = 13;

  // If set to true, the connection manager will use the real remote address

  // of the client connection when determining internal versus external origin and manipulating

  // various headers. If set to false or absent, the connection manager will use the

  // :ref:`config_http_conn_man_headers_x-forwarded-for` HTTP header. See the documentation for

  // :ref:`config_http_conn_man_headers_x-forwarded-for`,

  // :ref:`config_http_conn_man_headers_x-envoy-internal`, and

  // :ref:`config_http_conn_man_headers_x-envoy-external-address` for more information.

  google.protobuf.BoolValue use_remote_address = 14

      [(udpa.annotations.security).configure_for_untrusted_downstream = true];

  // The number of additional ingress proxy hops from the right side of the

  // :ref:`config_http_conn_man_headers_x-forwarded-for` HTTP header to trust when

  // determining the origin client's IP address. The default is zero if this option

  // is not specified. See the documentation for

  // :ref:`config_http_conn_man_headers_x-forwarded-for` for more information.

  uint32 xff_num_trusted_hops = 19;

  // Configures what network addresses are considered internal for stats and header sanitation

  // purposes. If unspecified, only RFC1918 IP addresses will be considered internal.

  // See the documentation for :ref:`config_http_conn_man_headers_x-envoy-internal` for more

  // information about internal/external addresses.

  InternalAddressConfig internal_address_config = 25;

  // If set, Envoy will not append the remote address to the

  // :ref:`config_http_conn_man_headers_x-forwarded-for` HTTP header. This may be used in

  // conjunction with HTTP filters that explicitly manipulate XFF after the HTTP connection manager

  // has mutated the request headers. While :ref:`use_remote_address

  // <envoy_api_field_extensions.filters.network.http_connection_manager.v3.HttpConnectionManager.use_remote_address>`

  // will also suppress XFF addition, it has consequences for logging and other

  // Envoy uses of the remote address, so *skip_xff_append* should be used

  // when only an elision of XFF addition is intended.

  bool skip_xff_append = 21;

  // Via header value to append to request and response headers. If this is

  // empty, no via header will be appended.

  string via = 22;

  // Whether the connection manager will generate the :ref:`x-request-id

  // <config_http_conn_man_headers_x-request-id>` header if it does not exist. This defaults to

  // true. Generating a random UUID4 is expensive so in high throughput scenarios where this feature

  // is not desired it can be disabled.

  google.protobuf.BoolValue generate_request_id = 15;

  // Whether the connection manager will keep the :ref:`x-request-id

  // <config_http_conn_man_headers_x-request-id>` header if passed for a request that is edge

  // (Edge request is the request from external clients to front Envoy) and not reset it, which

  // is the current Envoy behaviour. This defaults to false.

  bool preserve_external_request_id = 32;

  // If set, Envoy will always set :ref:`x-request-id <config_http_conn_man_headers_x-request-id>` header in response.

  // If this is false or not set, the request ID is returned in responses only if tracing is forced using

  // :ref:`x-envoy-force-trace <config_http_conn_man_headers_x-envoy-force-trace>` header.

  bool always_set_request_id_in_response = 37;

  // How to handle the :ref:`config_http_conn_man_headers_x-forwarded-client-cert` (XFCC) HTTP

  // header.

  ForwardClientCertDetails forward_client_cert_details = 16

      [(validate.rules).enum = {defined_only: true}];

  // This field is valid only when :ref:`forward_client_cert_details

  // <envoy_api_field_extensions.filters.network.http_connection_manager.v3.HttpConnectionManager.forward_client_cert_details>`

  // is APPEND_FORWARD or SANITIZE_SET and the client connection is mTLS. It specifies the fields in

  // the client certificate to be forwarded. Note that in the

  // :ref:`config_http_conn_man_headers_x-forwarded-client-cert` header, *Hash* is always set, and

  // *By* is always set when the client certificate presents the URI type Subject Alternative Name

  // value.

  SetCurrentClientCertDetails set_current_client_cert_details = 17;

  // If proxy_100_continue is true, Envoy will proxy incoming "Expect:

  // 100-continue" headers upstream, and forward "100 Continue" responses

  // downstream. If this is false or not set, Envoy will instead strip the

  // "Expect: 100-continue" header, and send a "100 Continue" response itself.

  bool proxy_100_continue = 18;

  // If

  // :ref:`use_remote_address

  // <envoy_api_field_extensions.filters.network.http_connection_manager.v3.HttpConnectionManager.use_remote_address>`

  // is true and represent_ipv4_remote_address_as_ipv4_mapped_ipv6 is true and the remote address is

  // an IPv4 address, the address will be mapped to IPv6 before it is appended to *x-forwarded-for*.

  // This is useful for testing compatibility of upstream services that parse the header value. For

  // example, 50.0.0.1 is represented as ::FFFF:50.0.0.1. See `IPv4-Mapped IPv6 Addresses

  // <https://tools.ietf.org/html/rfc4291#section-2.5.5.2>`_ for details. This will also affect the

  // :ref:`config_http_conn_man_headers_x-envoy-external-address` header. See

  // :ref:`http_connection_manager.represent_ipv4_remote_address_as_ipv4_mapped_ipv6

  // <config_http_conn_man_runtime_represent_ipv4_remote_address_as_ipv4_mapped_ipv6>` for runtime

  // control.

  // [#not-implemented-hide:]

  bool represent_ipv4_remote_address_as_ipv4_mapped_ipv6 = 20;

  repeated UpgradeConfig upgrade_configs = 23;

  // Should paths be normalized according to RFC 3986 before any processing of

  // requests by HTTP filters or routing? This affects the upstream *:path* header

  // as well. For paths that fail this check, Envoy will respond with 400 to

  // paths that are malformed. This defaults to false currently but will default

  // true in the future. When not specified, this value may be overridden by the

  // runtime variable

  // :ref:`http_connection_manager.normalize_path<config_http_conn_man_runtime_normalize_path>`.

  // See `Normalization and Comparison <https://tools.ietf.org/html/rfc3986#section-6>`_

  // for details of normalization.

  // Note that Envoy does not perform

  // `case normalization <https://tools.ietf.org/html/rfc3986#section-6.2.2.1>`_

  google.protobuf.BoolValue normalize_path = 30;

  // Determines if adjacent slashes in the path are merged into one before any processing of

  // requests by HTTP filters or routing. This affects the upstream *:path* header as well. Without

  // setting this option, incoming requests with path `//dir///file` will not match against route

  // with `prefix` match set to `/dir`. Defaults to `false`. Note that slash merging is not part of

  // `HTTP spec <https://tools.ietf.org/html/rfc3986>`_ and is provided for convenience.

  bool merge_slashes = 33;

  // The configuration of the request ID extension. This includes operations such as

  // generation, validation, and associated tracing operations.

  //

  // If not set, Envoy uses the default UUID-based behavior:

  //

  // 1. Request ID is propagated using *x-request-id* header.

  //

  // 2. Request ID is a universally unique identifier (UUID).

  //

  // 3. Tracing decision (sampled, forced, etc) is set in 14th byte of the UUID.

  RequestIDExtension request_id_extension = 36;

  // The configuration to customize local reply returned by Envoy. It can customize status code,

  // body text and response content type. If not specified, status code and text body are hard

  // coded in Envoy, the response content type is plain text.

  LocalReplyConfig local_reply_config = 38;

  // Determines if the port part should be removed from host/authority header before any processing

  // of request by HTTP filters or routing. The port would be removed only if it is equal to the :ref:`listener's<envoy_api_field_config.listener.v3.Listener.address>`

  // local port and request method is not CONNECT. This affects the upstream host header as well.

  // Without setting this option, incoming requests with host `example:443` will not match against

  // route with :ref:`domains<envoy_api_field_config.route.v3.VirtualHost.domains>` match set to `example`. Defaults to `false`. Note that port removal is not part

  // of `HTTP spec <https://tools.ietf.org/html/rfc3986>`_ and is provided for convenience.

  // Only one of `strip_matching_host_port` or `strip_any_host_port` can be set.

  bool strip_matching_host_port = 39

      [(udpa.annotations.field_migrate).oneof_promotion = "strip_port_mode"];

  oneof strip_port_mode {

    // Determines if the port part should be removed from host/authority header before any processing

    // of request by HTTP filters or routing. The port would be removed only if request method is not CONNECT.

    // This affects the upstream host header as well.

    // Without setting this option, incoming requests with host `example:443` will not match against

    // route with :ref:`domains<envoy_api_field_config.route.v3.VirtualHost.domains>` match set to `example`. Defaults to `false`. Note that port removal is not part

    // of `HTTP spec <https://tools.ietf.org/html/rfc3986>`_ and is provided for convenience.

    // Only one of `strip_matching_host_port` or `strip_any_host_port` can be set.

    bool strip_any_host_port = 42;

  }

  // Governs Envoy's behavior when receiving invalid HTTP from downstream.

  // If this option is false (default), Envoy will err on the conservative side handling HTTP

  // errors, terminating both HTTP/1.1 and HTTP/2 connections when receiving an invalid request.

  // If this option is set to true, Envoy will be more permissive, only resetting the invalid

  // stream in the case of HTTP/2 and leaving the connection open where possible (if the entire

  // request is read for HTTP/1.1)

  // In general this should be true for deployments receiving trusted traffic (L2 Envoys,

  // company-internal mesh) and false when receiving untrusted traffic (edge deployments).

  //

  // If different behaviors for invalid_http_message for HTTP/1 and HTTP/2 are

  // desired, one should use the new HTTP/1 option :ref:`override_stream_error_on_invalid_http_message

  // <envoy_v3_api_field_config.core.v3.Http1ProtocolOptions.override_stream_error_on_invalid_http_message>` or the new HTTP/2 option

  // :ref:`override_stream_error_on_invalid_http_message

  // <envoy_v3_api_field_config.core.v3.Http2ProtocolOptions.override_stream_error_on_invalid_http_message>`

  // *not* the deprecated but similarly named :ref:`stream_error_on_invalid_http_messaging

  // <envoy_v3_api_field_config.core.v3.Http2ProtocolOptions.stream_error_on_invalid_http_messaging>`

  google.protobuf.BoolValue stream_error_on_invalid_http_message = 40;

}

// The configuration to customize local reply returned by Envoy.

message LocalReplyConfig {

  // Configuration of list of mappers which allows to filter and change local response.

  // The mappers will be checked by the specified order until one is matched.

  repeated ResponseMapper mappers = 1;

  // The configuration to form response body from the :ref:`command operators <config_access_log_command_operators>`

  // and to specify response content type as one of: plain/text or application/json.

  //

  // Example one: "plain/text" ``body_format``.

  //

  // .. validated-code-block:: yaml

  //   :type-name: envoy.config.core.v3.SubstitutionFormatString

  //

  //   text_format: "%LOCAL_REPLY_BODY%:%RESPONSE_CODE%:path=%REQ(:path)%\n"

  //

  // The following response body in "plain/text" format will be generated for a request with

  // local reply body of "upstream connection error", response_code=503 and path=/foo.

  //

  // .. code-block:: text

  //

  //   upstream connect error:503:path=/foo

  //

  // Example two: "application/json" ``body_format``.

  //

  // .. validated-code-block:: yaml

  //   :type-name: envoy.config.core.v3.SubstitutionFormatString

  //

  //   json_format:

  //     status: "%RESPONSE_CODE%"

  //     message: "%LOCAL_REPLY_BODY%"

  //     path: "%REQ(:path)%"

  //

  // The following response body in "application/json" format would be generated for a request with

  // local reply body of "upstream connection error", response_code=503 and path=/foo.

  //

  // .. code-block:: json

  //

  //  {

  //    "status": 503,

  //    "message": "upstream connection error",

  //    "path": "/foo"

  //  }

  //

  config.core.v3.SubstitutionFormatString body_format = 2;

}

// The configuration to filter and change local response.

// [#next-free-field: 6]

message ResponseMapper {

  // Filter to determine if this mapper should apply.

  config.accesslog.v3.AccessLogFilter filter = 1 [(validate.rules).message = {required: true}];

  // The new response status code if specified.

  google.protobuf.UInt32Value status_code = 2 [(validate.rules).uint32 = {lt: 600 gte: 200}];

  // The new local reply body text if specified. It will be used in the `%LOCAL_REPLY_BODY%`

  // command operator in the `body_format`.

  config.core.v3.DataSource body = 3;

  // A per mapper `body_format` to override the :ref:`body_format <envoy_v3_api_field_extensions.filters.network.http_connection_manager.v3.LocalReplyConfig.body_format>`.

  // It will be used when this mapper is matched.

  config.core.v3.SubstitutionFormatString body_format_override = 4;

  // HTTP headers to add to a local reply. This allows the response mapper to append, to add

  // or to override headers of any local reply before it is sent to a downstream client.

  repeated config.core.v3.HeaderValueOption headers_to_add = 5

      [(validate.rules).repeated = {max_items: 1000}];

}

message Rds {

  option (udpa.annotations.versioning).previous_message_type =

      "envoy.config.filter.network.http_connection_manager.v2.Rds";

  // Configuration source specifier for RDS.

  config.core.v3.ConfigSource config_source = 1 [(validate.rules).message = {required: true}];

  // The name of the route configuration. This name will be passed to the RDS

  // API. This allows an Envoy configuration with multiple HTTP listeners (and

  // associated HTTP connection manager filters) to use different route

  // configurations.

  string route_config_name = 2;

}

// This message is used to work around the limitations with 'oneof' and repeated fields.

message ScopedRouteConfigurationsList {

  option (udpa.annotations.versioning).previous_message_type =

      "envoy.config.filter.network.http_connection_manager.v2.ScopedRouteConfigurationsList";

  repeated config.route.v3.ScopedRouteConfiguration scoped_route_configurations = 1

      [(validate.rules).repeated = {min_items: 1}];

}

// [#next-free-field: 6]

message ScopedRoutes {

  option (udpa.annotations.versioning).previous_message_type =

      "envoy.config.filter.network.http_connection_manager.v2.ScopedRoutes";

  // Specifies the mechanism for constructing "scope keys" based on HTTP request attributes. These

  // keys are matched against a set of :ref:`Key<envoy_api_msg_config.route.v3.ScopedRouteConfiguration.Key>`

  // objects assembled from :ref:`ScopedRouteConfiguration<envoy_api_msg_config.route.v3.ScopedRouteConfiguration>`

  // messages distributed via SRDS (the Scoped Route Discovery Service) or assigned statically via

  // :ref:`scoped_route_configurations_list<envoy_api_field_extensions.filters.network.http_connection_manager.v3.ScopedRoutes.scoped_route_configurations_list>`.

  //

  // Upon receiving a request's headers, the Router will build a key using the algorithm specified

  // by this message. This key will be used to look up the routing table (i.e., the

  // :ref:`RouteConfiguration<envoy_api_msg_config.route.v3.RouteConfiguration>`) to use for the request.

  message ScopeKeyBuilder {

    option (udpa.annotations.versioning).previous_message_type =

        "envoy.config.filter.network.http_connection_manager.v2.ScopedRoutes.ScopeKeyBuilder";

    // Specifies the mechanism for constructing key fragments which are composed into scope keys.

    message FragmentBuilder {

      option (udpa.annotations.versioning).previous_message_type =

          "envoy.config.filter.network.http_connection_manager.v2.ScopedRoutes.ScopeKeyBuilder."

          "FragmentBuilder";

      // Specifies how the value of a header should be extracted.

      // The following example maps the structure of a header to the fields in this message.

      //

      // .. code::

      //

      //              <0> <1>   <-- index

      //    X-Header: a=b;c=d

      //    |         || |

      //    |         || \----> <element_separator>

      //    |         ||

      //    |         |\----> <element.separator>

      //    |         |

      //    |         \----> <element.key>

      //    |

      //    \----> <name>

      //

      //    Each 'a=b' key-value pair constitutes an 'element' of the header field.

      message HeaderValueExtractor {

        option (udpa.annotations.versioning).previous_message_type =

            "envoy.config.filter.network.http_connection_manager.v2.ScopedRoutes.ScopeKeyBuilder."

            "FragmentBuilder.HeaderValueExtractor";

        // Specifies a header field's key value pair to match on.

        message KvElement {

          option (udpa.annotations.versioning).previous_message_type =

              "envoy.config.filter.network.http_connection_manager.v2.ScopedRoutes.ScopeKeyBuilder."

              "FragmentBuilder.HeaderValueExtractor.KvElement";

          // The separator between key and value (e.g., '=' separates 'k=v;...').

          // If an element is an empty string, the element is ignored.

          // If an element contains no separator, the whole element is parsed as key and the

          // fragment value is an empty string.

          // If there are multiple values for a matched key, the first value is returned.

          string separator = 1 [(validate.rules).string = {min_len: 1}];

          // The key to match on.

          string key = 2 [(validate.rules).string = {min_len: 1}];

        }

        // The name of the header field to extract the value from.

        //

        // .. note::

        //

        //   If the header appears multiple times only the first value is used.

        string name = 1 [(validate.rules).string = {min_len: 1}];

        // The element separator (e.g., ';' separates 'a;b;c;d').

        // Default: empty string. This causes the entirety of the header field to be extracted.

        // If this field is set to an empty string and 'index' is used in the oneof below, 'index'

        // must be set to 0.

        string element_separator = 2;

        oneof extract_type {

          // Specifies the zero based index of the element to extract.

          // Note Envoy concatenates multiple values of the same header key into a comma separated

          // string, the splitting always happens after the concatenation.

          uint32 index = 3;

          // Specifies the key value pair to extract the value from.

          KvElement element = 4;

        }

      }

      oneof type {

        option (validate.required) = true;

        // Specifies how a header field's value should be extracted.

        HeaderValueExtractor header_value_extractor = 1;

      }

    }

    // The final(built) scope key consists of the ordered union of these fragments, which are compared in order with the

    // fragments of a :ref:`ScopedRouteConfiguration<envoy_api_msg_config.route.v3.ScopedRouteConfiguration>`.

    // A missing fragment during comparison will make the key invalid, i.e., the computed key doesn't match any key.

    repeated FragmentBuilder fragments = 1 [(validate.rules).repeated = {min_items: 1}];

  }

  // The name assigned to the scoped routing configuration.

  string name = 1 [(validate.rules).string = {min_len: 1}];

  // The algorithm to use for constructing a scope key for each request.

  ScopeKeyBuilder scope_key_builder = 2 [(validate.rules).message = {required: true}];

  // Configuration source specifier for RDS.

  // This config source is used to subscribe to RouteConfiguration resources specified in

  // ScopedRouteConfiguration messages.

  config.core.v3.ConfigSource rds_config_source = 3 [(validate.rules).message = {required: true}];

  oneof config_specifier {

    option (validate.required) = true;

    // The set of routing scopes corresponding to the HCM. A scope is assigned to a request by

    // matching a key constructed from the request's attributes according to the algorithm specified

    // by the

    // :ref:`ScopeKeyBuilder<envoy_api_msg_extensions.filters.network.http_connection_manager.v3.ScopedRoutes.ScopeKeyBuilder>`

    // in this message.

    ScopedRouteConfigurationsList scoped_route_configurations_list = 4;

    // The set of routing scopes associated with the HCM will be dynamically loaded via the SRDS

    // API. A scope is assigned to a request by matching a key constructed from the request's

    // attributes according to the algorithm specified by the

    // :ref:`ScopeKeyBuilder<envoy_api_msg_extensions.filters.network.http_connection_manager.v3.ScopedRoutes.ScopeKeyBuilder>`

    // in this message.

    ScopedRds scoped_rds = 5;

  }

}

message ScopedRds {

  option (udpa.annotations.versioning).previous_message_type =

      "envoy.config.filter.network.http_connection_manager.v2.ScopedRds";

  // Configuration source specifier for scoped RDS.

  config.core.v3.ConfigSource scoped_rds_config_source = 1

      [(validate.rules).message = {required: true}];

}

// [#next-free-field: 6]

message HttpFilter {

  option (udpa.annotations.versioning).previous_message_type =

      "envoy.config.filter.network.http_connection_manager.v2.HttpFilter";

  reserved 3, 2;

  reserved "config";

  // The name of the filter configuration. The name is used as a fallback to

  // select an extension if the type of the configuration proto is not

  // sufficient. It also serves as a resource name in ExtensionConfigDS.

  string name = 1 [(validate.rules).string = {min_len: 1}];

  oneof config_type {

    // Filter specific configuration which depends on the filter being instantiated. See the supported

    // filters for further documentation.

    google.protobuf.Any typed_config = 4;

    // Configuration source specifier for an extension configuration discovery service.

    // In case of a failure and without the default configuration, the HTTP listener responds with code 500.

    // Extension configs delivered through this mechanism are not expected to require warming (see https://github.com/envoyproxy/envoy/issues/12061).

    config.core.v3.ExtensionConfigSource config_discovery = 5;

  }

}

message RequestIDExtension {

  option (udpa.annotations.versioning).previous_message_type =

      "envoy.config.filter.network.http_connection_manager.v2.RequestIDExtension";

  // Request ID extension specific configuration.

  google.protobuf.Any typed_config = 1;

}syntax = "proto3";

package envoy.extensions.filters.network.http_connection_manager.v3;

import "envoy/config/accesslog/v3/accesslog.proto";

import "envoy/config/core/v3/base.proto";

import "envoy/config/core/v3/config_source.proto";

import "envoy/config/core/v3/extension.proto";

import "envoy/config/core/v3/protocol.proto";

import "envoy/config/core/v3/substitution_format_string.proto";

import "envoy/config/route/v3/route.proto";

import "envoy/config/route/v3/scoped_route.proto";

import "envoy/config/trace/v3/http_tracer.proto";

import "envoy/type/tracing/v3/custom_tag.proto";

import "envoy/type/v3/percent.proto";

import "google/protobuf/any.proto";

import "google/protobuf/duration.proto";

import "google/protobuf/struct.proto";

import "google/protobuf/wrappers.proto";

import "envoy/annotations/deprecation.proto";

import "udpa/annotations/migrate.proto";

import "udpa/annotations/security.proto";

import "udpa/annotations/status.proto";

import "udpa/annotations/versioning.proto";

import "validate/validate.proto";

option java_package = "io.envoyproxy.envoy.extensions.filters.network.http_connection_manager.v3";

option java_outer_classname = "HttpConnectionManagerProto";

option java_multiple_files = true;

option (udpa.annotations.file_status).package_version_status = ACTIVE;

// [#protodoc-title: HTTP connection manager]

// HTTP connection manager :ref:`configuration overview <config_http_conn_man>`.

// [#extension: envoy.filters.network.http_connection_manager]

// [#next-free-field: 43]

message HttpConnectionManager {

  option (udpa.annotations.versioning).previous_message_type =

      "envoy.config.filter.network.http_connection_manager.v2.HttpConnectionManager";

  enum CodecType {

    // For every new connection, the connection manager will determine which

    // codec to use. This mode supports both ALPN for TLS listeners as well as

    // protocol inference for plaintext listeners. If ALPN data is available, it

    // is preferred, otherwise protocol inference is used. In almost all cases,

    // this is the right option to choose for this setting.

    AUTO = 0;

    // The connection manager will assume that the client is speaking HTTP/1.1.

    HTTP1 = 1;

    // The connection manager will assume that the client is speaking HTTP/2

    // (Envoy does not require HTTP/2 to take place over TLS or to use ALPN.

    // Prior knowledge is allowed).

    HTTP2 = 2;

    // [#not-implemented-hide:] QUIC implementation is not production ready yet. Use this enum with

    // caution to prevent accidental execution of QUIC code. I.e. `!= HTTP2` is no longer sufficient

    // to distinguish HTTP1 and HTTP2 traffic.

    HTTP3 = 3;

  }

  enum ServerHeaderTransformation {

    // Overwrite any Server header with the contents of server_name.

    OVERWRITE = 0;

    // If no Server header is present, append Server server_name

    // If a Server header is present, pass it through.

    APPEND_IF_ABSENT = 1;

    // Pass through the value of the server header, and do not append a header

    // if none is present.

    PASS_THROUGH = 2;

  }

  // How to handle the :ref:`config_http_conn_man_headers_x-forwarded-client-cert` (XFCC) HTTP

  // header.

  enum ForwardClientCertDetails {

    // Do not send the XFCC header to the next hop. This is the default value.

    SANITIZE = 0;

    // When the client connection is mTLS (Mutual TLS), forward the XFCC header

    // in the request.

    FORWARD_ONLY = 1;

    // When the client connection is mTLS, append the client certificate

    // information to the requestâ€™s XFCC header and forward it.

    APPEND_FORWARD = 2;

    // When the client connection is mTLS, reset the XFCC header with the client

    // certificate information and send it to the next hop.

    SANITIZE_SET = 3;

    // Always forward the XFCC header in the request, regardless of whether the

    // client connection is mTLS.

    ALWAYS_FORWARD_ONLY = 4;

  }

  // [#next-free-field: 10]

  message Tracing {

    option (udpa.annotations.versioning).previous_message_type =

        "envoy.config.filter.network.http_connection_manager.v2.HttpConnectionManager.Tracing";

    enum OperationName {

      // The HTTP listener is used for ingress/incoming requests.

      INGRESS = 0;

      // The HTTP listener is used for egress/outgoing requests.

      EGRESS = 1;

    }

    reserved 1, 2;

    reserved "operation_name", "request_headers_for_tags";

    // Target percentage of requests managed by this HTTP connection manager that will be force

    // traced if the :ref:`x-client-trace-id <config_http_conn_man_headers_x-client-trace-id>`

    // header is set. This field is a direct analog for the runtime variable

    // 'tracing.client_sampling' in the :ref:`HTTP Connection Manager

    // <config_http_conn_man_runtime>`.

    // Default: 100%

    type.v3.Percent client_sampling = 3;

    // Target percentage of requests managed by this HTTP connection manager that will be randomly

    // selected for trace generation, if not requested by the client or not forced. This field is

    // a direct analog for the runtime variable 'tracing.random_sampling' in the

    // :ref:`HTTP Connection Manager <config_http_conn_man_runtime>`.

    // Default: 100%

    type.v3.Percent random_sampling = 4;

    // Target percentage of requests managed by this HTTP connection manager that will be traced

    // after all other sampling checks have been applied (client-directed, force tracing, random

    // sampling). This field functions as an upper limit on the total configured sampling rate. For

    // instance, setting client_sampling to 100% but overall_sampling to 1% will result in only 1%

    // of client requests with the appropriate headers to be force traced. This field is a direct

    // analog for the runtime variable 'tracing.global_enabled' in the

    // :ref:`HTTP Connection Manager <config_http_conn_man_runtime>`.

    // Default: 100%

    type.v3.Percent overall_sampling = 5;

    // Whether to annotate spans with additional data. If true, spans will include logs for stream

    // events.

    bool verbose = 6;

    // Maximum length of the request path to extract and include in the HttpUrl tag. Used to

    // truncate lengthy request paths to meet the needs of a tracing backend.

    // Default: 256

    google.protobuf.UInt32Value max_path_tag_length = 7;

    // A list of custom tags with unique tag name to create tags for the active span.

    repeated type.tracing.v3.CustomTag custom_tags = 8;

    // Configuration for an external tracing provider.

    // If not specified, no tracing will be performed.

    //

    // .. attention::

    //   Please be aware that *envoy.tracers.opencensus* provider can only be configured once

    //   in Envoy lifetime.

    //   Any attempts to reconfigure it or to use different configurations for different HCM filters

    //   will be rejected.

    //   Such a constraint is inherent to OpenCensus itself. It cannot be overcome without changes

    //   on OpenCensus side.

    config.trace.v3.Tracing.Http provider = 9;

  }

  message InternalAddressConfig {

    option (udpa.annotations.versioning).previous_message_type =

        "envoy.config.filter.network.http_connection_manager.v2.HttpConnectionManager."

        "InternalAddressConfig";

    // Whether unix socket addresses should be considered internal.

    bool unix_sockets = 1;

  }

  // [#next-free-field: 7]

  message SetCurrentClientCertDetails {

    option (udpa.annotations.versioning).previous_message_type =

        "envoy.config.filter.network.http_connection_manager.v2.HttpConnectionManager."

        "SetCurrentClientCertDetails";

    reserved 2;

    // Whether to forward the subject of the client cert. Defaults to false.

    google.protobuf.BoolValue subject = 1;

    // Whether to forward the entire client cert in URL encoded PEM format. This will appear in the

    // XFCC header comma separated from other values with the value Cert="PEM".

    // Defaults to false.

    bool cert = 3;

    // Whether to forward the entire client cert chain (including the leaf cert) in URL encoded PEM

    // format. This will appear in the XFCC header comma separated from other values with the value

    // Chain="PEM".

    // Defaults to false.

    bool chain = 6;

    // Whether to forward the DNS type Subject Alternative Names of the client cert.

    // Defaults to false.

    bool dns = 4;

    // Whether to forward the URI type Subject Alternative Name of the client cert. Defaults to

    // false.

    bool uri = 5;

  }

  // The configuration for HTTP upgrades.

  // For each upgrade type desired, an UpgradeConfig must be added.

  //

  // .. warning::

  //

  //    The current implementation of upgrade headers does not handle

  //    multi-valued upgrade headers. Support for multi-valued headers may be

  //    added in the future if needed.

  //

  // .. warning::

  //    The current implementation of upgrade headers does not work with HTTP/2

  //    upstreams.

  message UpgradeConfig {

    option (udpa.annotations.versioning).previous_message_type =

        "envoy.config.filter.network.http_connection_manager.v2.HttpConnectionManager."

        "UpgradeConfig";

    // The case-insensitive name of this upgrade, e.g. "websocket".

    // For each upgrade type present in upgrade_configs, requests with

    // Upgrade: [upgrade_type]

    // will be proxied upstream.

    string upgrade_type = 1;

    // If present, this represents the filter chain which will be created for

    // this type of upgrade. If no filters are present, the filter chain for

    // HTTP connections will be used for this upgrade type.

    repeated HttpFilter filters = 2;

    // Determines if upgrades are enabled or disabled by default. Defaults to true.

    // This can be overridden on a per-route basis with :ref:`cluster

    // <envoy_api_field_config.route.v3.RouteAction.upgrade_configs>` as documented in the

    // :ref:`upgrade documentation <arch_overview_upgrades>`.

    google.protobuf.BoolValue enabled = 3;

  }

  reserved 27, 11;

  reserved "idle_timeout";

  // Supplies the type of codec that the connection manager should use.

  CodecType codec_type = 1 [(validate.rules).enum = {defined_only: true}];

  // The human readable prefix to use when emitting statistics for the

  // connection manager. See the :ref:`statistics documentation <config_http_conn_man_stats>` for

  // more information.

  string stat_prefix = 2 [(validate.rules).string = {min_len: 1}];

  oneof route_specifier {

    option (validate.required) = true;

    // The connection managerâ€™s route table will be dynamically loaded via the RDS API.

    Rds rds = 3;

    // The route table for the connection manager is static and is specified in this property.

    config.route.v3.RouteConfiguration route_config = 4;

    // A route table will be dynamically assigned to each request based on request attributes

    // (e.g., the value of a header). The "routing scopes" (i.e., route tables) and "scope keys" are

    // specified in this message.

    ScopedRoutes scoped_routes = 31;

  }

  // A list of individual HTTP filters that make up the filter chain for

  // requests made to the connection manager. :ref:`Order matters <arch_overview_http_filters_ordering>`

  // as the filters are processed sequentially as request events happen.

  repeated HttpFilter http_filters = 5;

  // Whether the connection manager manipulates the :ref:`config_http_conn_man_headers_user-agent`

  // and :ref:`config_http_conn_man_headers_downstream-service-cluster` headers. See the linked

  // documentation for more information. Defaults to false.

  google.protobuf.BoolValue add_user_agent = 6;

  // Presence of the object defines whether the connection manager

  // emits :ref:`tracing <arch_overview_tracing>` data to the :ref:`configured tracing provider

  // <envoy_api_msg_config.trace.v3.Tracing>`.

  Tracing tracing = 7;

  // Additional settings for HTTP requests handled by the connection manager. These will be

  // applicable to both HTTP1 and HTTP2 requests.

  config.core.v3.HttpProtocolOptions common_http_protocol_options = 35

      [(udpa.annotations.security).configure_for_untrusted_downstream = true];

  // Additional HTTP/1 settings that are passed to the HTTP/1 codec.

  config.core.v3.Http1ProtocolOptions http_protocol_options = 8;

  // Additional HTTP/2 settings that are passed directly to the HTTP/2 codec.

  config.core.v3.Http2ProtocolOptions http2_protocol_options = 9

      [(udpa.annotations.security).configure_for_untrusted_downstream = true];

  // An optional override that the connection manager will write to the server

  // header in responses. If not set, the default is *envoy*.

  string server_name = 10

      [(validate.rules).string = {well_known_regex: HTTP_HEADER_VALUE strict: false}];

  // Defines the action to be applied to the Server header on the response path.

  // By default, Envoy will overwrite the header with the value specified in

  // server_name.

  ServerHeaderTransformation server_header_transformation = 34

      [(validate.rules).enum = {defined_only: true}];

  // The maximum request headers size for incoming connections.

  // If unconfigured, the default max request headers allowed is 60 KiB.

  // Requests that exceed this limit will receive a 431 response.

  // The max configurable limit is 96 KiB, based on current implementation

  // constraints.

  google.protobuf.UInt32Value max_request_headers_kb = 29

      [(validate.rules).uint32 = {lte: 96 gt: 0}];

  // The stream idle timeout for connections managed by the connection manager.

  // If not specified, this defaults to 5 minutes. The default value was selected

  // so as not to interfere with any smaller configured timeouts that may have

  // existed in configurations prior to the introduction of this feature, while

  // introducing robustness to TCP connections that terminate without a FIN.

  //

  // This idle timeout applies to new streams and is overridable by the

  // :ref:`route-level idle_timeout

  // <envoy_api_field_config.route.v3.RouteAction.idle_timeout>`. Even on a stream in

  // which the override applies, prior to receipt of the initial request

  // headers, the :ref:`stream_idle_timeout

  // <envoy_api_field_extensions.filters.network.http_connection_manager.v3.HttpConnectionManager.stream_idle_timeout>`

  // applies. Each time an encode/decode event for headers or data is processed

  // for the stream, the timer will be reset. If the timeout fires, the stream

  // is terminated with a 408 Request Timeout error code if no upstream response

  // header has been received, otherwise a stream reset occurs.

  //

  // This timeout also specifies the amount of time that Envoy will wait for the peer to open enough

  // window to write any remaining stream data once the entirety of stream data (local end stream is

  // true) has been buffered pending available window. In other words, this timeout defends against

  // a peer that does not release enough window to completely write the stream, even though all

  // data has been proxied within available flow control windows. If the timeout is hit in this

  // case, the :ref:`tx_flush_timeout <config_http_conn_man_stats_per_codec>` counter will be

  // incremented. Note that :ref:`max_stream_duration

  // <envoy_api_field_config.core.v3.HttpProtocolOptions.max_stream_duration>` does not apply to

  // this corner case.

  //

  // If the :ref:`overload action <config_overload_manager_overload_actions>` "envoy.overload_actions.reduce_timeouts"

  // is configured, this timeout is scaled according to the value for

  // :ref:`HTTP_DOWNSTREAM_STREAM_IDLE <envoy_api_enum_value_config.overload.v3.ScaleTimersOverloadActionConfig.TimerType.HTTP_DOWNSTREAM_STREAM_IDLE>`.

  //

  // Note that it is possible to idle timeout even if the wire traffic for a stream is non-idle, due

  // to the granularity of events presented to the connection manager. For example, while receiving

  // very large request headers, it may be the case that there is traffic regularly arriving on the

  // wire while the connection manage is only able to observe the end-of-headers event, hence the

  // stream may still idle timeout.

  //

  // A value of 0 will completely disable the connection manager stream idle

  // timeout, although per-route idle timeout overrides will continue to apply.

  google.protobuf.Duration stream_idle_timeout = 24

      [(udpa.annotations.security).configure_for_untrusted_downstream = true];

  // The amount of time that Envoy will wait for the entire request to be received.

  // The timer is activated when the request is initiated, and is disarmed when the last byte of the

  // request is sent upstream (i.e. all decoding filters have processed the request), OR when the

  // response is initiated. If not specified or set to 0, this timeout is disabled.

  google.protobuf.Duration request_timeout = 28

      [(udpa.annotations.security).configure_for_untrusted_downstream = true];

  // The amount of time that Envoy will wait for the request headers to be received. The timer is

  // activated when the first byte of the headers is received, and is disarmed when the last byte of

  // the headers has been received. If not specified or set to 0, this timeout is disabled.

  google.protobuf.Duration request_headers_timeout = 41 [

    (validate.rules).duration = {gte {}},

    (udpa.annotations.security).configure_for_untrusted_downstream = true

  ];

  // The time that Envoy will wait between sending an HTTP/2 â€œshutdown

  // notificationâ€ (GOAWAY frame with max stream ID) and a final GOAWAY frame.

  // This is used so that Envoy provides a grace period for new streams that

  // race with the final GOAWAY frame. During this grace period, Envoy will

  // continue to accept new streams. After the grace period, a final GOAWAY

  // frame is sent and Envoy will start refusing new streams. Draining occurs

  // both when a connection hits the idle timeout or during general server

  // draining. The default grace period is 5000 milliseconds (5 seconds) if this

  // option is not specified.

  google.protobuf.Duration drain_timeout = 12;

  // The delayed close timeout is for downstream connections managed by the HTTP connection manager.

  // It is defined as a grace period after connection close processing has been locally initiated

  // during which Envoy will wait for the peer to close (i.e., a TCP FIN/RST is received by Envoy

  // from the downstream connection) prior to Envoy closing the socket associated with that

  // connection.

  // NOTE: This timeout is enforced even when the socket associated with the downstream connection

  // is pending a flush of the write buffer. However, any progress made writing data to the socket

  // will restart the timer associated with this timeout. This means that the total grace period for

  // a socket in this state will be

  // <total_time_waiting_for_write_buffer_flushes>+<delayed_close_timeout>.

  //

  // Delaying Envoy's connection close and giving the peer the opportunity to initiate the close

  // sequence mitigates a race condition that exists when downstream clients do not drain/process

  // data in a connection's receive buffer after a remote close has been detected via a socket

  // write(). This race leads to such clients failing to process the response code sent by Envoy,

  // which could result in erroneous downstream processing.

  //

  // If the timeout triggers, Envoy will close the connection's socket.

  //

  // The default timeout is 1000 ms if this option is not specified.

  //

  // .. NOTE::

  //    To be useful in avoiding the race condition described above, this timeout must be set

  //    to *at least* <max round trip time expected between clients and Envoy>+<100ms to account for

  //    a reasonable "worst" case processing time for a full iteration of Envoy's event loop>.

  //

  // .. WARNING::

  //    A value of 0 will completely disable delayed close processing. When disabled, the downstream

  //    connection's socket will be closed immediately after the write flush is completed or will

  //    never close if the write flush does not complete.

  google.protobuf.Duration delayed_close_timeout = 26;

  // Configuration for :ref:`HTTP access logs <arch_overview_access_logs>`

  // emitted by the connection manager.

  repeated config.accesslog.v3.AccessLog access_log = 13;

  // If set to true, the connection manager will use the real remote address

  // of the client connection when determining internal versus external origin and manipulating

  // various headers. If set to false or absent, the connection manager will use the

  // :ref:`config_http_conn_man_headers_x-forwarded-for` HTTP header. See the documentation for

  // :ref:`config_http_conn_man_headers_x-forwarded-for`,

  // :ref:`config_http_conn_man_headers_x-envoy-internal`, and

  // :ref:`config_http_conn_man_headers_x-envoy-external-address` for more information.

  google.protobuf.BoolValue use_remote_address = 14

      [(udpa.annotations.security).configure_for_untrusted_downstream = true];

  // The number of additional ingress proxy hops from the right side of the

  // :ref:`config_http_conn_man_headers_x-forwarded-for` HTTP header to trust when

  // determining the origin client's IP address. The default is zero if this option

  // is not specified. See the documentation for

  // :ref:`config_http_conn_man_headers_x-forwarded-for` for more information.

  uint32 xff_num_trusted_hops = 19;

  // Configures what network addresses are considered internal for stats and header sanitation

  // purposes. If unspecified, only RFC1918 IP addresses will be considered internal.

  // See the documentation for :ref:`config_http_conn_man_headers_x-envoy-internal` for more

  // information about internal/external addresses.

  InternalAddressConfig internal_address_config = 25;

  // If set, Envoy will not append the remote address to the

  // :ref:`config_http_conn_man_headers_x-forwarded-for` HTTP header. This may be used in

  // conjunction with HTTP filters that explicitly manipulate XFF after the HTTP connection manager

  // has mutated the request headers. While :ref:`use_remote_address

  // <envoy_api_field_extensions.filters.network.http_connection_manager.v3.HttpConnectionManager.use_remote_address>`

  // will also suppress XFF addition, it has consequences for logging and other

  // Envoy uses of the remote address, so *skip_xff_append* should be used

  // when only an elision of XFF addition is intended.

  bool skip_xff_append = 21;

  // Via header value to append to request and response headers. If this is

  // empty, no via header will be appended.

  string via = 22;

  // Whether the connection manager will generate the :ref:`x-request-id

  // <config_http_conn_man_headers_x-request-id>` header if it does not exist. This defaults to

  // true. Generating a random UUID4 is expensive so in high throughput scenarios where this feature

  // is not desired it can be disabled.

  google.protobuf.BoolValue generate_request_id = 15;

  // Whether the connection manager will keep the :ref:`x-request-id

  // <config_http_conn_man_headers_x-request-id>` header if passed for a request that is edge

  // (Edge request is the request from external clients to front Envoy) and not reset it, which

  // is the current Envoy behaviour. This defaults to false.

  bool preserve_external_request_id = 32;

  // If set, Envoy will always set :ref:`x-request-id <config_http_conn_man_headers_x-request-id>` header in response.

  // If this is false or not set, the request ID is returned in responses only if tracing is forced using

  // :ref:`x-envoy-force-trace <config_http_conn_man_headers_x-envoy-force-trace>` header.

  bool always_set_request_id_in_response = 37;

  // How to handle the :ref:`config_http_conn_man_headers_x-forwarded-client-cert` (XFCC) HTTP

  // header.

  ForwardClientCertDetails forward_client_cert_details = 16

      [(validate.rules).enum = {defined_only: true}];

  // This field is valid only when :ref:`forward_client_cert_details

  // <envoy_api_field_extensions.filters.network.http_connection_manager.v3.HttpConnectionManager.forward_client_cert_details>`

  // is APPEND_FORWARD or SANITIZE_SET and the client connection is mTLS. It specifies the fields in

  // the client certificate to be forwarded. Note that in the

  // :ref:`config_http_conn_man_headers_x-forwarded-client-cert` header, *Hash* is always set, and

  // *By* is always set when the client certificate presents the URI type Subject Alternative Name

  // value.

  SetCurrentClientCertDetails set_current_client_cert_details = 17;

  // If proxy_100_continue is true, Envoy will proxy incoming "Expect:

  // 100-continue" headers upstream, and forward "100 Continue" responses

  // downstream. If this is false or not set, Envoy will instead strip the

  // "Expect: 100-continue" header, and send a "100 Continue" response itself.

  bool proxy_100_continue = 18;

  // If

  // :ref:`use_remote_address

  // <envoy_api_field_extensions.filters.network.http_connection_manager.v3.HttpConnectionManager.use_remote_address>`

  // is true and represent_ipv4_remote_address_as_ipv4_mapped_ipv6 is true and the remote address is

  // an IPv4 address, the address will be mapped to IPv6 before it is appended to *x-forwarded-for*.

  // This is useful for testing compatibility of upstream services that parse the header value. For

  // example, 50.0.0.1 is represented as ::FFFF:50.0.0.1. See `IPv4-Mapped IPv6 Addresses

  // <https://tools.ietf.org/html/rfc4291#section-2.5.5.2>`_ for details. This will also affect the

  // :ref:`config_http_conn_man_headers_x-envoy-external-address` header. See

  // :ref:`http_connection_manager.represent_ipv4_remote_address_as_ipv4_mapped_ipv6

  // <config_http_conn_man_runtime_represent_ipv4_remote_address_as_ipv4_mapped_ipv6>` for runtime

  // control.

  // [#not-implemented-hide:]

  bool represent_ipv4_remote_address_as_ipv4_mapped_ipv6 = 20;

  repeated UpgradeConfig upgrade_configs = 23;

  // Should paths be normalized according to RFC 3986 before any processing of

  // requests by HTTP filters or routing? This affects the upstream *:path* header

  // as well. For paths that fail this check, Envoy will respond with 400 to

  // paths that are malformed. This defaults to false currently but will default

  // true in the future. When not specified, this value may be overridden by the

  // runtime variable

  // :ref:`http_connection_manager.normalize_path<config_http_conn_man_runtime_normalize_path>`.

  // See `Normalization and Comparison <https://tools.ietf.org/html/rfc3986#section-6>`_

  // for details of normalization.

  // Note that Envoy does not perform

  // `case normalization <https://tools.ietf.org/html/rfc3986#section-6.2.2.1>`_

  google.protobuf.BoolValue normalize_path = 30;

  // Determines if adjacent slashes in the path are merged into one before any processing of

  // requests by HTTP filters or routing. This affects the upstream *:path* header as well. Without

  // setting this option, incoming requests with path `//dir///file` will not match against route

  // with `prefix` match set to `/dir`. Defaults to `false`. Note that slash merging is not part of

  // `HTTP spec <https://tools.ietf.org/html/rfc3986>`_ and is provided for convenience.

  bool merge_slashes = 33;

  // The configuration of the request ID extension. This includes operations such as

  // generation, validation, and associated tracing operations.

  //

  // If not set, Envoy uses the default UUID-based behavior:

  //

  // 1. Request ID is propagated using *x-request-id* header.

  //

  // 2. Request ID is a universally unique identifier (UUID).

  //

  // 3. Tracing decision (sampled, forced, etc) is set in 14th byte of the UUID.

  RequestIDExtension request_id_extension = 36;

  // The configuration to customize local reply returned by Envoy. It can customize status code,

  // body text and response content type. If not specified, status code and text body are hard

  // coded in Envoy, the response content type is plain text.

  LocalReplyConfig local_reply_config = 38;

  // Determines if the port part should be removed from host/authority header before any processing

  // of request by HTTP filters or routing. The port would be removed only if it is equal to the :ref:`listener's<envoy_api_field_config.listener.v3.Listener.address>`

  // local port and request method is not CONNECT. This affects the upstream host header as well.

  // Without setting this option, incoming requests with host `example:443` will not match against

  // route with :ref:`domains<envoy_api_field_config.route.v3.VirtualHost.domains>` match set to `example`. Defaults to `false`. Note that port removal is not part

  // of `HTTP spec <https://tools.ietf.org/html/rfc3986>`_ and is provided for convenience.

  // Only one of `strip_matching_host_port` or `strip_any_host_port` can be set.

  bool strip_matching_host_port = 39

      [(udpa.annotations.field_migrate).oneof_promotion = "strip_port_mode"];

  oneof strip_port_mode {

    // Determines if the port part should be removed from host/authority header before any processing

    // of request by HTTP filters or routing. The port would be removed only if request method is not CONNECT.

    // This affects the upstream host header as well.

    // Without setting this option, incoming requests with host `example:443` will not match against

    // route with :ref:`domains<envoy_api_field_config.route.v3.VirtualHost.domains>` match set to `example`. Defaults to `false`. Note that port removal is not part

    // of `HTTP spec <https://tools.ietf.org/html/rfc3986>`_ and is provided for convenience.

    // Only one of `strip_matching_host_port` or `strip_any_host_port` can be set.

    bool strip_any_host_port = 42;

  }

  // Governs Envoy's behavior when receiving invalid HTTP from downstream.

  // If this option is false (default), Envoy will err on the conservative side handling HTTP

  // errors, terminating both HTTP/1.1 and HTTP/2 connections when receiving an invalid request.

  // If this option is set to true, Envoy will be more permissive, only resetting the invalid

  // stream in the case of HTTP/2 and leaving the connection open where possible (if the entire

  // request is read for HTTP/1.1)

  // In general this should be true for deployments receiving trusted traffic (L2 Envoys,

  // company-internal mesh) and false when receiving untrusted traffic (edge deployments).

  //

  // If different behaviors for invalid_http_message for HTTP/1 and HTTP/2 are

  // desired, one should use the new HTTP/1 option :ref:`override_stream_error_on_invalid_http_message

  // <envoy_v3_api_field_config.core.v3.Http1ProtocolOptions.override_stream_error_on_invalid_http_message>` or the new HTTP/2 option

  // :ref:`override_stream_error_on_invalid_http_message

  // <envoy_v3_api_field_config.core.v3.Http2ProtocolOptions.override_stream_error_on_invalid_http_message>`

  // *not* the deprecated but similarly named :ref:`stream_error_on_invalid_http_messaging

  // <envoy_v3_api_field_config.core.v3.Http2ProtocolOptions.stream_error_on_invalid_http_messaging>`

  google.protobuf.BoolValue stream_error_on_invalid_http_message = 40;

}

// The configuration to customize local reply returned by Envoy.

message LocalReplyConfig {

  // Configuration of list of mappers which allows to filter and change local response.

  // The mappers will be checked by the specified order until one is matched.

  repeated ResponseMapper mappers = 1;

  // The configuration to form response body from the :ref:`command operators <config_access_log_command_operators>`

  // and to specify response content type as one of: plain/text or application/json.

  //

  // Example one: "plain/text" ``body_format``.

  //

  // .. validated-code-block:: yaml

  //   :type-name: envoy.config.core.v3.SubstitutionFormatString

  //

  //   text_format: "%LOCAL_REPLY_BODY%:%RESPONSE_CODE%:path=%REQ(:path)%\n"

  //

  // The following response body in "plain/text" format will be generated for a request with

  // local reply body of "upstream connection error", response_code=503 and path=/foo.

  //

  // .. code-block:: text

  //

  //   upstream connect error:503:path=/foo

  //

  // Example two: "application/json" ``body_format``.

  //

  // .. validated-code-block:: yaml

  //   :type-name: envoy.config.core.v3.SubstitutionFormatString

  //

  //   json_format:

  //     status: "%RESPONSE_CODE%"

  //     message: "%LOCAL_REPLY_BODY%"

  //     path: "%REQ(:path)%"

  //

  // The following response body in "application/json" format would be generated for a request with

  // local reply body of "upstream connection error", response_code=503 and path=/foo.

  //

  // .. code-block:: json

  //

  //  {

  //    "status": 503,

  //    "message": "upstream connection error",

  //    "path": "/foo"

  //  }

  //

  config.core.v3.SubstitutionFormatString body_format = 2;

}

// The configuration to filter and change local response.

// [#next-free-field: 6]

message ResponseMapper {

  // Filter to determine if this mapper should apply.

  config.accesslog.v3.AccessLogFilter filter = 1 [(validate.rules).message = {required: true}];

  // The new response status code if specified.

  google.protobuf.UInt32Value status_code = 2 [(validate.rules).uint32 = {lt: 600 gte: 200}];

  // The new local reply body text if specified. It will be used in the `%LOCAL_REPLY_BODY%`

  // command operator in the `body_format`.

  config.core.v3.DataSource body = 3;

  // A per mapper `body_format` to override the :ref:`body_format <envoy_v3_api_field_extensions.filters.network.http_connection_manager.v3.LocalReplyConfig.body_format>`.

  // It will be used when this mapper is matched.

  config.core.v3.SubstitutionFormatString body_format_override = 4;

  // HTTP headers to add to a local reply. This allows the response mapper to append, to add

  // or to override headers of any local reply before it is sent to a downstream client.

  repeated config.core.v3.HeaderValueOption headers_to_add = 5

      [(validate.rules).repeated = {max_items: 1000}];

}

message Rds {

  option (udpa.annotations.versioning).previous_message_type =

      "envoy.config.filter.network.http_connection_manager.v2.Rds";

  // Configuration source specifier for RDS.

  config.core.v3.ConfigSource config_source = 1 [(validate.rules).message = {required: true}];

  // The name of the route configuration. This name will be passed to the RDS

  // API. This allows an Envoy configuration with multiple HTTP listeners (and

  // associated HTTP connection manager filters) to use different route

  // configurations.

  string route_config_name = 2;

}

// This message is used to work around the limitations with 'oneof' and repeated fields.

message ScopedRouteConfigurationsList {

  option (udpa.annotations.versioning).previous_message_type =

      "envoy.config.filter.network.http_connection_manager.v2.ScopedRouteConfigurationsList";

  repeated config.route.v3.ScopedRouteConfiguration scoped_route_configurations = 1

      [(validate.rules).repeated = {min_items: 1}];

}

// [#next-free-field: 6]

message ScopedRoutes {

  option (udpa.annotations.versioning).previous_message_type =

      "envoy.config.filter.network.http_connection_manager.v2.ScopedRoutes";

  // Specifies the mechanism for constructing "scope keys" based on HTTP request attributes. These

  // keys are matched against a set of :ref:`Key<envoy_api_msg_config.route.v3.ScopedRouteConfiguration.Key>`

  // objects assembled from :ref:`ScopedRouteConfiguration<envoy_api_msg_config.route.v3.ScopedRouteConfiguration>`

  // messages distributed via SRDS (the Scoped Route Discovery Service) or assigned statically via

  // :ref:`scoped_route_configurations_list<envoy_api_field_extensions.filters.network.http_connection_manager.v3.ScopedRoutes.scoped_route_configurations_list>`.

  //

  // Upon receiving a request's headers, the Router will build a key using the algorithm specified

  // by this message. This key will be used to look up the routing table (i.e., the

  // :ref:`RouteConfiguration<envoy_api_msg_config.route.v3.RouteConfiguration>`) to use for the request.

  message ScopeKeyBuilder {

    option (udpa.annotations.versioning).previous_message_type =

        "envoy.config.filter.network.http_connection_manager.v2.ScopedRoutes.ScopeKeyBuilder";

    // Specifies the mechanism for constructing key fragments which are composed into scope keys.

    message FragmentBuilder {

      option (udpa.annotations.versioning).previous_message_type =

          "envoy.config.filter.network.http_connection_manager.v2.ScopedRoutes.ScopeKeyBuilder."

          "FragmentBuilder";

      // Specifies how the value of a header should be extracted.

      // The following example maps the structure of a header to the fields in this message.

      //

      // .. code::

      //

      //              <0> <1>   <-- index

      //    X-Header: a=b;c=d

      //    |         || |

      //    |         || \----> <element_separator>

      //    |         ||

      //    |         |\----> <element.separator>

      //    |         |

      //    |         \----> <element.key>

      //    |

      //    \----> <name>

      //

      //    Each 'a=b' key-value pair constitutes an 'element' of the header field.

      message HeaderValueExtractor {

        option (udpa.annotations.versioning).previous_message_type =

            "envoy.config.filter.network.http_connection_manager.v2.ScopedRoutes.ScopeKeyBuilder."

            "FragmentBuilder.HeaderValueExtractor";

        // Specifies a header field's key value pair to match on.

        message KvElement {

          option (udpa.annotations.versioning).previous_message_type =

              "envoy.config.filter.network.http_connection_manager.v2.ScopedRoutes.ScopeKeyBuilder."

              "FragmentBuilder.HeaderValueExtractor.KvElement";

          // The separator between key and value (e.g., '=' separates 'k=v;...').

          // If an element is an empty string, the element is ignored.

          // If an element contains no separator, the whole element is parsed as key and the

          // fragment value is an empty string.

          // If there are multiple values for a matched key, the first value is returned.

          string separator = 1 [(validate.rules).string = {min_len: 1}];

          // The key to match on.

          string key = 2 [(validate.rules).string = {min_len: 1}];

        }

        // The name of the header field to extract the value from.

        //

        // .. note::

        //

        //   If the header appears multiple times only the first value is used.

        string name = 1 [(validate.rules).string = {min_len: 1}];

        // The element separator (e.g., ';' separates 'a;b;c;d').

        // Default: empty string. This causes the entirety of the header field to be extracted.

        // If this field is set to an empty string and 'index' is used in the oneof below, 'index'

        // must be set to 0.

        string element_separator = 2;

        oneof extract_type {

          // Specifies the zero based index of the element to extract.

          // Note Envoy concatenates multiple values of the same header key into a comma separated

          // string, the splitting always happens after the concatenation.

          uint32 index = 3;

          // Specifies the key value pair to extract the value from.

          KvElement element = 4;

        }

      }

      oneof type {

        option (validate.required) = true;

        // Specifies how a header field's value should be extracted.

        HeaderValueExtractor header_value_extractor = 1;

      }

    }

    // The final(built) scope key consists of the ordered union of these fragments, which are compared in order with the

    // fragments of a :ref:`ScopedRouteConfiguration<envoy_api_msg_config.route.v3.ScopedRouteConfiguration>`.

    // A missing fragment during comparison will make the key invalid, i.e., the computed key doesn't match any key.

    repeated FragmentBuilder fragments = 1 [(validate.rules).repeated = {min_items: 1}];

  }

  // The name assigned to the scoped routing configuration.

  string name = 1 [(validate.rules).string = {min_len: 1}];

  // The algorithm to use for constructing a scope key for each request.

  ScopeKeyBuilder scope_key_builder = 2 [(validate.rules).message = {required: true}];

  // Configuration source specifier for RDS.

  // This config source is used to subscribe to RouteConfiguration resources specified in

  // ScopedRouteConfiguration messages.

  config.core.v3.ConfigSource rds_config_source = 3 [(validate.rules).message = {required: true}];

  oneof config_specifier {

    option (validate.required) = true;

    // The set of routing scopes corresponding to the HCM. A scope is assigned to a request by

    // matching a key constructed from the request's attributes according to the algorithm specified

    // by the

    // :ref:`ScopeKeyBuilder<envoy_api_msg_extensions.filters.network.http_connection_manager.v3.ScopedRoutes.ScopeKeyBuilder>`

    // in this message.

    ScopedRouteConfigurationsList scoped_route_configurations_list = 4;

    // The set of routing scopes associated with the HCM will be dynamically loaded via the SRDS

    // API. A scope is assigned to a request by matching a key constructed from the request's

    // attributes according to the algorithm specified by the

    // :ref:`ScopeKeyBuilder<envoy_api_msg_extensions.filters.network.http_connection_manager.v3.ScopedRoutes.ScopeKeyBuilder>`

    // in this message.

    ScopedRds scoped_rds = 5;

  }

}

message ScopedRds {

  option (udpa.annotations.versioning).previous_message_type =

      "envoy.config.filter.network.http_connection_manager.v2.ScopedRds";

  // Configuration source specifier for scoped RDS.

  config.core.v3.ConfigSource scoped_rds_config_source = 1

      [(validate.rules).message = {required: true}];

}

// [#next-free-field: 6]

message HttpFilter {

  option (udpa.annotations.versioning).previous_message_type =

      "envoy.config.filter.network.http_connection_manager.v2.HttpFilter";

  reserved 3, 2;

  reserved "config";

  // The name of the filter configuration. The name is used as a fallback to

  // select an extension if the type of the configuration proto is not

  // sufficient. It also serves as a resource name in ExtensionConfigDS.

  string name = 1 [(validate.rules).string = {min_len: 1}];

  oneof config_type {

    // Filter specific configuration which depends on the filter being instantiated. See the supported

    // filters for further documentation.

    google.protobuf.Any typed_config = 4;

    // Configuration source specifier for an extension configuration discovery service.

    // In case of a failure and without the default configuration, the HTTP listener responds with code 500.

    // Extension configs delivered through this mechanism are not expected to require warming (see https://github.com/envoyproxy/envoy/issues/12061).

    config.core.v3.ExtensionConfigSource config_discovery = 5;

  }

}

message RequestIDExtension {

  option (udpa.annotations.versioning).previous_message_type =

      "envoy.config.filter.network.http_connection_manager.v2.RequestIDExtension";

  // Request ID extension specific configuration.

  google.protobuf.Any typed_config = 1;

}