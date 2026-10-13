<!--
LAB STARTER ONLY — validate against your Wazuh version and decoded event fields.
Back up local_rules.xml before merging. Do not assume this is production-ready.
Test with authorized lab telemetry and verify frequency/correlation behavior.
-->
<group name="authentication,windows,bruteforce,">
  <rule id="100100" level="10" frequency="5" timeframe="60">
    <if_matched_sid>60122</if_matched_sid>
    <description>LAB: repeated Windows failed logons; validate parent rule and correlation behavior.</description>
    <mitre>
      <id>T1110</id>
    </mitre>
  </rule>
</group>
