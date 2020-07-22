See https://github.com/p4lang/p4c/pull/2474

# Steps to test checksum for Sara

1. Add test-tcp3-bmv2.p4 and test-tcp3-bmv2.stf from this repo to your
   p4c/testdata/p4_16_samples/ directory.
2. Go to p4c/build and issue the cmake cmd: `cmake -DCMAKE_BUILD_TYPE=DEBUG ..`
   3. The cmake cmd adds a new test to p4c called `test-tcp3-bmv2.p4.test`
   4. Run the test as follows:
      `~/p4c/build$ bmv2/testdata/p4_16_samples/test-tcp3-bmv2.p4.test -v`
   You will see the test passes.
   5. Open test-tcp3-bmv2.stf in an editor and copy its `expect` pkt.
   6. Load pkt in Scapy using `p = Raw("")` where the `expect` pkt data is
      inserted between double-quotes.  Then, run `hexdump(p)` in Scapy
	  and save the output to a file, say, `foo`.
   7. Load `foo` as hexadecimal in Wireshark "Import From Hex Dump".
      Select TCP in Encapsulation portion of the Dialog box. Provide port
	  80 for TCP destination port.
   8. When the pkt is imported successfully in Wireshark, see that the TCP
      checksum is correct.
