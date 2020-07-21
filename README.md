# Work in Progress for TCP update_checksum in P4-16

## Following diffs show P4 code completed so far

```P4
diff --git a/testdata/p4_16_samples/checksum1-bmv2.p4 b/testdata/p4_16_samples/checksum1-bmv2.p4
index a4fb9557..d9e9be49 100644
--- a/testdata/p4_16_samples/checksum1-bmv2.p4
+++ b/testdata/p4_16_samples/checksum1-bmv2.p4
@@ -37,7 +37,7 @@ header tcp_t {
     bit<3>  ecn;
     bit<6>  ctrl;
     bit<16> window;
-    bit<16> checksum;
+    bit<16> checksum; // Includes Pseudo Hdr + TCP segment (hdr + payload)
     bit<16> urgentPtr;
 }
 
@@ -46,10 +46,15 @@ header IPv4_up_to_ihl_only_h {
     bit<4>       ihl;
 }
 
+header payload_t {
+    varbit<1496> text;
+}
+
 struct headers {
     ethernet_t    ethernet;
     ipv4_t        ipv4;
     tcp_t         tcp;
+    payload_t     pyld;
 }
 
 struct mystruct1_t {
@@ -59,6 +64,7 @@ struct mystruct1_t {
 
 struct metadata {
     mystruct1_t mystruct1;
+    bit<16>     tcpLen; // includes TCP hdr len + TCP payload len
 }
 
 typedef tuple<
@@ -80,6 +86,7 @@ parser parserI(packet_in pkt,
                inout metadata meta,
                inout standard_metadata_t stdmeta)
 {
+    bit<32> varLen;
     state start {
         pkt.extract(hdr.ethernet);
         transition select(hdr.ethernet.etherType) {
@@ -108,6 +115,9 @@ parser parserI(packet_in pkt,
     }
     state parse_tcp {
         pkt.extract(hdr.tcp);
+        meta.tcpLen = hdr.ipv4.totalLen - (bit<16>)(hdr.ipv4.ihl*4);
+        varLen = (bit<32>)(8*meta.tcpLen - hdr.tcp.minSizeInBits());
+        pkt.extract(hdr.pyld, varLen);
         transition accept;
     }
 }
@@ -191,6 +201,27 @@ control uc(inout headers hdr,
                 hdr.ipv4.options
             },
             hdr.ipv4.hdrChecksum, HashAlgorithm.csum16);
+
+        update_checksum(hdr.tcp.isValid(),
+            { hdr.ipv4.srcAddr,
+              hdr.ipv4.dstAddr,
+              8w0,
+              hdr.ipv4.protocol,
+              meta.tcpLen,
+              hdr.tcp.srcPort,
+              hdr.tcp.dstPort,
+              hdr.tcp.seqNo,
+              hdr.tcp.ackNo,
+              hdr.tcp.dataOffset,
+              hdr.tcp.res,
+              hdr.tcp.ecn,
+              hdr.tcp.ctrl,
+              hdr.tcp.window,
+              16w0, // checksum
+              hdr.tcp.urgentPtr,
+              hdr.pyld.text
+	    },
+            hdr.tcp.checksum, HashAlgorithm.csum16);
     }
 }
```
