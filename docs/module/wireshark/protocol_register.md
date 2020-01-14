# protocol register

## a21

```cpp
    proto_a21 = proto_register_protocol("A21 Protocol", "A21", "a21");

void proto_reg_handoff_a21(void)
{
    dissector_handle_t a21_handle;

    a21_handle = create_dissector_handle(dissect_a21, proto_a21);
    gcsna_handle = find_dissector_add_dependency("gcsna", proto_a21);
    dissector_add_uint_with_preference("udp.port", A21_PORT, a21_handle);
}
```

## aarp

```cpp
  proto_aarp = proto_register_protocol("Appletalk Address Resolution Protocol",
                                       "AARP",
                                       "aarp");
void
proto_reg_handoff_aarp(void)
{
  dissector_handle_t aarp_handle;

  aarp_handle = create_dissector_handle(dissect_aarp, proto_aarp);
  dissector_add_uint("ethertype", ETHERTYPE_AARP, aarp_handle);
  dissector_add_uint("chdlc.protocol", ETHERTYPE_AARP, aarp_handle);
}
```

## aasp

```cpp
    proto_aasp = proto_register_protocol("Aastra Signalling Protocol", "AASP", "aasp");

void
proto_reg_handoff_aasp(void)
{
    dissector_handle_t aasp_handle;
    aasp_handle = create_dissector_handle(dissect_aasp, proto_aasp);
    dissector_add_string("media_type", "message/x-aasp-signalling", aasp_handle);
}
```

## acap

```cpp
    proto_acap = proto_register_protocol("Application Configuration Access Protocol",
                         "ACAP", "acap");
void
proto_reg_handoff_acap(void)
{
    dissector_add_uint_with_preference("tcp.port", TCP_PORT_ACAP, acap_handle);
}
```

## acn

- [ ] test

```cpp
  proto_acn = proto_register_protocol (
    "Architecture for Control Networks", /* name */
    "ACN",                               /* short name */
    "acn"                                /* abbrev */
    );
void
proto_reg_handoff_acn(void)
{
  /* dissector_handle_t acn_handle; */
  /* acn_handle = create_dissector_handle(dissect_acn, proto_acn); */
  /* dissector_add_for_decode_as_with_preference("udp.port", acn_handle);                         */
  heur_dissector_add("udp", dissect_acn_heur, "ACN over UDP", "acn_udp", proto_acn, HEURISTIC_DISABLE);
}
```

## acp133

- [ ] test

```cpp
#define PNAME  "ACP133 Attribute Syntaxes"
#define PSNAME "ACP133"
#define PFNAME "acp133"

'ber.oid'
```

## acr122

- [ ] test

## acse

- [ ] test

```cpp
#define PNAME  "ISO 8650-1 OSI Association Control Service"
#define PSNAME "ACSE"
#define PFNAME "acse"

#define CLPNAME  "ISO 10035-1 OSI Connectionless Association Control Service"
#define CLPSNAME "CLACSE"
#define CLPFNAME "clacse"

'ber.oid'
```

## actrace

```cpp
    /* Register protocol */
    proto_actrace = proto_register_protocol("AudioCodes Trunk Trace", "ACtrace", "actrace");

/* The registration hand-off routine */
void proto_reg_handoff_actrace(void)
{
    dissector_handle_t actrace_handle;

    /* Get a handle for the lapd dissector. */
    lapd_handle = find_dissector_add_dependency("lapd", proto_actrace);

    actrace_handle = create_dissector_handle(dissect_actrace, proto_actrace);
    dissector_add_uint_with_preference("udp.port", UDP_PORT_ACTRACE, actrace_handle);
}
```

## adb_cs

```cpp
    proto_adb_cs = proto_register_protocol("Android Debug Bridge Client-Server", "ADB CS", "adb_cs");

void
proto_reg_handoff_adb_cs(void)
{
    adb_service_handle = find_dissector_add_dependency("adb_service", proto_adb_cs);

    dissector_add_for_decode_as_with_preference("tcp.port", adb_cs_handle);
}
```

## adb

```cpp
    proto_adb = proto_register_protocol("Android Debug Bridge", "ADB", "adb");

void
proto_reg_handoff_adb(void)
{
    adb_service_handle = find_dissector_add_dependency("adb_service", proto_adb);

    dissector_add_for_decode_as_with_preference("tcp.port",     adb_handle);
    dissector_add_for_decode_as("usb.device",   adb_handle);
    dissector_add_for_decode_as("usb.product",  adb_handle);
    dissector_add_for_decode_as("usb.protocol", adb_handle);

    proto_tcp = proto_get_id_by_filter_name("tcp");
    proto_usb = proto_get_id_by_filter_name("usb");
}

```

## adwin_config

- [ ] test

```cpp
    proto_adwin_config =
        proto_register_protocol("ADwin configuration protocol",
                    "ADwin-Config", "adwin_config");

    /* Required function calls to register the header fields and
        subtrees used */
    proto_register_field_array(proto_adwin_config, hf, array_length(hf));
    proto_register_subtree_array(ett, array_length(ett));
}

void
proto_reg_handoff_adwin_config(void)
{
    heur_dissector_add("udp", dissect_adwin_config_udp, "ADwin-Config over UDP", "adwin_config_udp", proto_adwin_config, HEURISTIC_ENABLE);
    heur_dissector_add("tcp", dissect_adwin_config_tcp, "ADwin-Config over TCP", "adwin_config_tcp", proto_adwin_config, HEURISTIC_ENABLE);
}
```

## adwin

```cpp
void
proto_reg_handoff_adwin(void)
{
    dissector_handle_t adwin_handle;

    adwin_handle = create_dissector_handle(dissect_adwin, proto_adwin);
    dissector_add_uint_with_preference("udp.port", ADWIN_COMM_PORT, adwin_handle);
}

    proto_adwin = proto_register_protocol("ADwin communication protocol",
                          "ADwin", "adwin");
```

## aeron

```cpp
    proto_aeron = proto_register_protocol("Aeron Protocol", "Aeron", "aeron");

void proto_reg_handoff_aeron(void)
{
    aeron_dissector_handle = create_dissector_handle(dissect_aeron, proto_aeron);
    dissector_add_for_decode_as_with_preference("udp.port", aeron_dissector_handle);
    heur_dissector_add("udp", test_aeron_packet, "Aeron over UDP", "aeron_udp", proto_aeron, HEURISTIC_DISABLE);
}
```

## afp

- [ ] test

```cpp
    proto_afp = proto_register_protocol("Apple Filing Protocol", "AFP", "afp");
void
proto_reg_handoff_afp(void)
{
    spotlight_handle = find_dissector_add_dependency("afp_spotlight", proto_afp);
}
```

## afs

```cpp
    proto_afs = proto_register_protocol("Andrew File System (AFS)",
        "AFS (RX)", "afs");
```

## agentx

```cpp
    proto_agentx = proto_register_protocol("AgentX", "AgentX", "agentx");

void
proto_reg_handoff_agentx(void)
{
    dissector_handle_t agentx_handle;
    agentx_handle = create_dissector_handle(dissect_agentx, proto_agentx);
    dissector_add_uint_with_preference("tcp.port", AGENTX_TCP_PORT, agentx_handle);
}
```

## aim

```cpp
    proto_aim = proto_register_protocol("AOL Instant Messenger", "AIM", "aim");

    dissector_add_uint_range_with_preference("tcp.port", TCP_PORTS_AIM_DEFAULT, aim_handle);

```

## ain

```cpp

#define PNAME  "Advanced Intelligent Network"
#define PSNAME "AIN"
#define PFNAME "ain"
```

## ajp13

```cpp
  proto_ajp13 = proto_register_protocol("Apache JServ Protocol v1.3", "AJP13", "ajp13");

void
proto_reg_handoff_ajp13(void)
{
  dissector_handle_t ajp13_handle;
  ajp13_handle = create_dissector_handle(dissect_ajp13, proto_ajp13);
  dissector_add_uint_with_preference("tcp.port", AJP13_TCP_PORT, ajp13_handle);
}
```

## alcap

- [ ] alcap

```cpp
    proto_alcap = proto_register_protocol(alcap_proto_name, alcap_proto_name_short, "alcap");
void
proto_reg_handoff_alcap(void)
{
    dissector_add_uint("mtp3.service_indicator", MTP_SI_AAL2, alcap_handle);
}
```

## AllJoyn

- [ ] ajns
- [ ] aj
- [x] ardp

```cpp
    proto_AllJoyn_ns = proto_register_protocol("AllJoyn Name Service Protocol", "AllJoyn NS", "ajns");

    /* Message protocols */
    proto_AllJoyn_mess = proto_register_protocol("AllJoyn Message Protocol", "AllJoyn", "aj");

    proto_register_field_array(proto_AllJoyn_ns, hf, array_length(hf));
    proto_register_subtree_array(ett, array_length(ett));
    expert_alljoyn = expert_register_protocol(proto_AllJoyn_mess);
    expert_register_field_array(expert_alljoyn, ei, array_length(ei));

    /* ARDP */                        /* name, short name, abbrev */
    proto_AllJoyn_ardp = proto_register_protocol("AllJoyn Reliable Datagram Protocol", "AllJoyn ARDP", "ardp");
}

void
proto_reg_handoff_AllJoyn(void)
{
    dissector_handle_t alljoyn_handle_ns;
    dissector_handle_t alljoyn_handle_ardp;

    alljoyn_handle_ns = create_dissector_handle(dissect_AllJoyn_name_server, proto_AllJoyn_ns);
    alljoyn_handle_ardp = create_dissector_handle(dissect_AllJoyn_ardp, proto_AllJoyn_ardp);
    dissector_add_uint_with_preference("tcp.port", ALLJOYN_NAME_SERVER_PORT, alljoyn_handle_ns);
    dissector_add_uint_with_preference("tcp.port", ALLJOYN_MESSAGE_PORT,    );

    dissector_add_uint_with_preference("udp.port", ALLJOYN_NAME_SERVER_PORT, alljoyn_handle_ns);

    /* The ARDP dissector will directly call the AllJoyn message dissector if needed.
     * This includes the case where there is no ARDP data. */
    dissector_add_uint_with_preference("udp.port", ALLJOYN_MESSAGE_PORT, alljoyn_handle_ardp);
}
```

## amp

- [ ] amp

```cpp
    proto_amp = proto_register_protocol("AMP", "AMP", "amp");

void
proto_reg_handoff_amp(void)
{
    dissector_add_uint("ccsds.apid", AMP_APID, amp_handle);
    dissector_add_for_decode_as_with_preference("udp.port", amp_handle);
}
"ccsds.apid"
```

## amqp

```cpp
    proto_amqp = proto_register_protocol("Advanced Message Queueing Protocol", "AMQP", "amqp");

void
proto_reg_handoff_amqp(void)
{
    static guint old_amqps_port = 0;
    static gboolean initialize = FALSE;

    if (!initialize) {
        /* Register TCP port for dissection */
        dissector_add_uint_with_preference("tcp.port", AMQP_PORT, amqp_tcp_handle);

        dissector_add_uint("amqp.version", AMQP_V0_9, create_dissector_handle( dissect_amqpv0_9, proto_amqpv0_9 ));
        dissector_add_uint("amqp.version", AMQP_V0_10, create_dissector_handle( dissect_amqpv0_10, proto_amqpv0_10 ));
        dissector_add_uint("amqp.version", AMQP_V1_0, create_dissector_handle( dissect_amqpv1_0, proto_amqpv1_0 ));

        initialize = TRUE;
    }

    /* Register for SSL/TLS payload dissection */
    if (old_amqps_port != amqps_port) {
        if (old_amqps_port != 0)
            ssl_dissector_delete(old_amqps_port, amqp_tcp_handle);
        ssl_dissector_add(amqps_port, amqp_tcp_handle);
        old_amqps_port = amqps_port;
    }

    media_type_subdissector_table = find_dissector_table ("media_type");
}
"tcp.port"
"tls.port"
```

## amr

- [ ] amr

```cpp
    proto_amr = proto_register_protocol("Adaptive Multi-Rate","AMR", "amr");

"rtp_dyn_payload_type"
"h245.gef.name"
```

## amt

```cpp
    proto_amt = proto_register_protocol("Automatic Multicast Tunneling", "AMT", "amt");

void
proto_reg_handoff_amt(void)
{
    dissector_handle_t amt_handle;

    ip_handle = find_dissector_add_dependency("ip", proto_amt);

    amt_handle = create_dissector_handle(dissect_amt, proto_amt);
    dissector_add_uint_with_preference("udp.port", AMT_UDP_PORT, amt_handle);
}
```

## ancp

```cpp
    proto_ancp = proto_register_protocol (
            "Access Node Control Protocol",
            "ANCP",
            "ancp"
            );

void
proto_reg_handoff_ancp(void)
{
    dissector_handle_t ancp_handle;

    ancp_handle = create_dissector_handle(dissect_ancp, proto_ancp);
    dissector_add_uint_with_preference("tcp.port", ANCP_PORT, ancp_handle);
    stats_tree_register("ancp", "ancp", "ANCP", 0,
            ancp_stats_tree_packet, ancp_stats_tree_init, NULL);
}
```

## ans

```cpp
    proto_ans = proto_register_protocol("Intel ANS probe", "ANS", "ans");

void
proto_reg_handoff_ans(void)
{
    dissector_handle_t ans_handle;

    ans_handle = create_dissector_handle(dissect_ans, proto_ans);
    dissector_add_uint("ethertype", ETHERTYPE_INTEL_ANS, ans_handle);
}
```

## ansi_637

- [x] ansi_637_trans
- [ ] ansi_637_tele

```cpp
    proto_ansi_637_tele =
        proto_register_protocol(ansi_proto_name_tele, "ANSI IS-637-A Teleservice", "ansi_637_tele");
    proto_ansi_637_trans =
        proto_register_protocol(ansi_proto_name_trans, "ANSI IS-637-A Transport", "ansi_637_trans");

    ansi_637_trans_app_handle = create_dissector_handle(dissect_ansi_637_trans_app, proto_ansi_637_trans);
    dissector_add_string("media_type", "application/vnd.3gpp2.sms", ansi_637_trans_app_handle);
```

## ansi_*

pass

## aodv

```cpp
    proto_aodv = proto_register_protocol("Ad hoc On-demand Distance Vector Routing Protocol", "AODV", "aodv");

void
proto_reg_handoff_aodv(void)
{
    dissector_handle_t aodv_handle;

    aodv_handle = create_dissector_handle(dissect_aodv,
                                              proto_aodv);
    dissector_add_uint_with_preference("udp.port", UDP_PORT_AODV, aodv_handle);
}
```

## aoe

```cpp
  proto_aoe = proto_register_protocol("ATAoverEthernet", "AOE", "aoe");

void
proto_reg_handoff_aoe(void)
{
  dissector_add_uint("ethertype", ETHERTYPE_AOE, aoe_handle);
}
```

## aol

```cpp
    proto_aol = proto_register_protocol("America Online","AOL","aol");

void proto_reg_handoff_aol(void) {
    dissector_handle_t aol_handle;

    aol_handle = create_dissector_handle(dissect_aol,proto_aol);
    dissector_add_uint_with_preference("tcp.port",AOL_PORT,aol_handle);
}
```

## ap1394

- [ ] ap1394

```cpp
  proto_ap1394 = proto_register_protocol("Apple IP-over-IEEE 1394", "IP/IEEE1394", "ap1394");

void
proto_reg_handoff_ap1394(void)
{
  dissector_handle_t ap1394_handle;
  capture_dissector_handle_t ap1394_cap_handle;

  ethertype_subdissector_table = find_dissector_table("ethertype");

  ap1394_handle = create_dissector_handle(dissect_ap1394, proto_ap1394);
  dissector_add_uint("wtap_encap", WTAP_ENCAP_APPLE_IP_OVER_IEEE1394, ap1394_handle);

  ap1394_cap_handle = create_capture_dissector_handle(capture_ap1394, proto_ap1394);
  capture_dissector_add_uint("wtap_encap", WTAP_ENCAP_APPLE_IP_OVER_IEEE1394, ap1394_cap_handle);
}
"wtap_encap"
```

## app-pkix-cert

```cpp
        proto_cert = proto_register_protocol(
                        "PKIX CERT File Format",
                        "PKIX Certificate",
                        "pkix-cert"
        );
void
proto_reg_handoff_cert(void)
{
        dissector_handle_t cert_handle;

        cert_handle = create_dissector_handle(dissect_cert, proto_cert);

        /* Register the PKIX-CERT media type */
        dissector_add_string("media_type", "application/pkix-cert", cert_handle);
}
```

## applemidi

- [ ] applemidi

```cpp
#define APPLEMIDI_DISSECTOR_NAME                "Apple Network-MIDI Session Protocol"
#define APPLEMIDI_DISSECTOR_SHORTNAME           "AppleMIDI"
#define APPLEMIDI_DISSECTOR_ABBREVIATION        "applemidi"

void
proto_reg_handoff_applemidi( void ) {


    applemidi_handle = create_dissector_handle( dissect_applemidi, proto_applemidi );

    rtp_handle = find_dissector_add_dependency( "rtp", proto_applemidi );
    heur_dissector_add( "udp", dissect_applemidi_heur, "Apple MIDI over UDP", "applemidi_udp", proto_applemidi, HEURISTIC_ENABLE );
}

```

## aprs

```cpp
    proto_aprs = proto_register_protocol("Automatic Position Reporting System", "APRS", "aprs");
```

## ar_drone

```cpp
    proto_ar_drone = proto_register_protocol("AR Drone Packet", "AR Drone", "ar_drone");
void
proto_reg_handoff_ar_drone(void)
{
    dissector_handle_t ar_drone_handle;

    ar_drone_handle = create_dissector_handle(dissect_ar_drone, proto_ar_drone);

    heur_dissector_add("udp", dissect_ar_drone, "AR Drone over UDP", "ar_drone_udp", proto_ar_drone, HEURISTIC_ENABLE);
    dissector_add_for_decode_as_with_preference("udp.port", ar_drone_handle);
}
```

## arcnet

- [ ] arcent

```cpp
  proto_arcnet = proto_register_protocol ("ARCNET", "ARCNET", "arcnet");

void
proto_reg_handoff_arcnet (void)
{
  dissector_handle_t arcnet_handle, arcnet_linux_handle;
  capture_dissector_handle_t arcnet_cap_handle;

  arcnet_handle = create_dissector_handle (dissect_arcnet, proto_arcnet);
  dissector_add_uint ("wtap_encap", WTAP_ENCAP_ARCNET, arcnet_handle);

  arcnet_linux_handle = create_dissector_handle (dissect_arcnet_linux, proto_arcnet);
  dissector_add_uint ("wtap_encap", WTAP_ENCAP_ARCNET_LINUX, arcnet_linux_handle);

  proto_ipx = proto_get_id_by_filter_name("ipx");

  arcnet_cap_handle = create_capture_dissector_handle(capture_arcnet, proto_arcnet);
  capture_dissector_add_uint("wtap_encap", WTAP_ENCAP_ARCNET_LINUX, arcnet_cap_handle);
  arcnet_cap_handle = create_capture_dissector_handle(capture_arcnet_has_exception, proto_arcnet);
  capture_dissector_add_uint("wtap_encap", WTAP_ENCAP_ARCNET, arcnet_cap_handle);

  ip_cap_handle = find_capture_dissector("ip");
  arp_cap_handle = find_capture_dissector("arp");
}
"wtap_encap"
```

## armagetronad

```cpp
    proto_armagetronad = proto_register_protocol("The Armagetron Advanced OpenGL Tron clone", "Armagetronad", "armagetronad");

void proto_reg_handoff_armagetronad(void)
{
    dissector_add_uint_range_with_preference("udp.port", ARMAGETRONAD_UDP_PORT_RANGE, armagetronad_handle);
}
```

## arp

- [ ] arp

```cpp
  proto_arp = proto_register_protocol("Address Resolution Protocol",
                                      "ARP/RARP", "arp");
```

## artemis

```cpp
    proto_artemis = proto_register_protocol ( "Artemis Core Protocol", "Artemis", "artemis" );

void
proto_reg_handoff_artemis(void)
{
    static gboolean initialize = FALSE;

    if (!initialize) {
        /* Register TCP port for dissection */
        dissector_add_uint_with_preference("tcp.port", ARTEMIS_PORT, artemis_tcp_handle);
        initialize = TRUE;
    }
}
```

## artnet

- [ ] artnet

```cpp
  proto_artnet = proto_register_protocol("Art-Net", "ARTNET", "artnet");
  proto_register_field_array(proto_artnet, hf, array_length(hf));
  proto_register_subtree_array(ett, array_length(ett));
}

void
proto_reg_handoff_artnet(void) {
  dissector_handle_t artnet_handle;

  artnet_handle   = create_dissector_handle(dissect_artnet, proto_artnet);
  dissector_add_for_decode_as_with_preference("udp.port", artnet_handle);
  rdm_handle      = find_dissector_add_dependency("rdm", proto_artnet);
  dmx_chan_handle = find_dissector_add_dependency("dmx-chan", proto_artnet);

  heur_dissector_add("udp", dissect_artnet_heur, "ARTNET over UDP", "artnet_udp", proto_artnet, HEURISTIC_ENABLE);
}
```

## aruba-adp

```cpp

    proto_aruba_adp = proto_register_protocol("Aruba Discovery Protocol",
                                        "ADP", "adp");
void
proto_reg_handoff_aruba_adp(void)
{
    dissector_handle_t adp_handle;

    adp_handle = create_dissector_handle(dissect_aruba_adp, proto_aruba_adp);
    dissector_add_uint_with_preference("udp.port", UDP_PORT_ADP, adp_handle);
}
```

## aruba_erm

```CPP
    proto_aruba_erm = proto_register_protocol(PROTO_LONG_NAME, "ARUBA_ERM" , "aruba_erm");

"udp.port"
?
```

## aruba_iap

```cpp
    proto_aruba_iap = proto_register_protocol("Aruba Instant AP Protocol",
                    "aruba_iap", "aruba_iap");
void
proto_reg_handoff_aruba_iap(void)
{
    dissector_handle_t iap_handle;

    iap_handle = create_dissector_handle(dissect_aruba_iap, proto_aruba_iap);
    dissector_add_uint("ethertype", ETHERTYPE_IAP, iap_handle);
}
```

## papi

```cpp
    proto_papi = proto_register_protocol("Aruba PAPI", "PAPI", "papi");

void
proto_reg_handoff_papi(void)
{
    dissector_handle_t papi_handle;

    papi_handle = create_dissector_handle(dissect_papi, proto_papi);
    dissector_add_uint_with_preference("udp.port", UDP_PORT_PAPI, papi_handle);
}
```
