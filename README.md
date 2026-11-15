UWF-DIAG v1.0.5 - UNIFIED WRITE FILTER DIAGNOSTICS
==================================================

PURPOSE
-------
UWF-DIAG v1.0.5 retains Windows Firewall / Windows Filtering
Platform (WFP) diagnostics and read-only section for Microsoft's
Unified Write Filter (UWF).

This is a diagnostic collector. It does NOT enable or disable UWF, protect or
unprotect volumes, add/remove file or registry exclusions, change overlay
sizes or thresholds, change servicing mode, change services, alter audit
policy, or modify firewall/WFP rules.

IMPORTANT DISTINCTION
---------------------
Unified Write Filter is NOT Windows Defender Firewall and is NOT a network
packet firewall. UWF protects selected storage volumes. It intercepts writes
to protected volumes and redirects those writes into a virtual overlay (RAM
or disk, depending on configuration). If the overlay approaches its warning
or critical threshold, UWF raises events. If the overlay reaches exhaustion,
write operations can fail and Windows may become sluggish or unresponsive.

WHAT THE NEW SECTION COLLECTS
-----------------------------
Section 11: MICROSOFT UNIFIED WRITE FILTER (UWF) - READ ONLY

1. Optional feature state
   Uses DISM /Online /Get-FeatureInfo for Client-UnifiedWriteFilter. This is a
   read-only feature query. It records Enabled, Disabled, pending, or query
   failure where DISM reports a state.

2. UWF WMI provider availability
   Enumerates UWF_* class definitions in root\standardcimv2\embedded. It
   does not invoke WMI methods. Class discovery helps distinguish a supported
   provider from an unavailable namespace/provider.

3. Driver and service inventory
   Collects current Win32_SystemDriver and Win32_Service records whose name
   or display name identifies UWF. This is current-state inventory only.

4. uwfmgr read-only configuration commands
   Runs only these information-retrieval commands, when uwfmgr.exe exists:
     uwfmgr.exe get-config
     uwfmgr.exe volume get-config all
     uwfmgr.exe overlay get-config
     uwfmgr.exe overlay get-available-space
     uwfmgr.exe overlay get-consumption
     uwfmgr.exe file get-exclusions all
     uwfmgr.exe registry get-exclusions
     uwfmgr.exe servicing get-config
   The configuration commands show current-session and next-session values
   where supported by uwfmgr. The output and exit code are retained. No
   mutating uwfmgr verbs (Enable, Disable, Protect, Unprotect, Add-Exclusion,
   Remove-Exclusion, Set-Size, Set-Threshold, Servicing Enable/Disable, etc.)
   are executed.

5. UWF event evidence
   Checks the System log for provider uwfvol event IDs 1, 2, and 3 over the
   selected lookback period:
     ID 1 = overlay warning threshold reached
     ID 2 = overlay critical threshold reached
     ID 3 = overlay returned to normal
   It also checks the documented UnifiedWriteFilter Admin and Operational
   channels, plus any discoverable channels matching UnifiedWriteFilter.
   Admin generally carries error messages; Operational generally carries
   informational/configuration/servicing messages. Disabled logs are NOT
   enabled by this diagnostic.

NEW COMMAND-LINE PARAMETER
--------------------------
-MaxUwfEvents (default 2000; allowed range 100-20000)
Maximum events collected per UWF event log. This is independent of
-MaxAuditEvents, which limits the WFP 5152/5157 collection.

OUTPUT FILES ADDED
------------------
UWF_OptionalFeature.txt          DISM feature state query
UWF_WMI_Classes.csv              UWF WMI class names and class metadata
UWF_DriversAndServices.csv       UWF-related driver/service inventory
UWF_GetConfig.txt                Global UWF current/next session configuration
UWF_Volumes.txt                  All volume protection configuration
UWF_OverlayConfig.txt            Overlay type, size and threshold configuration
UWF_OverlayAvailableSpace.txt    Overlay capacity available now
UWF_OverlayConsumption.txt       Overlay consumption now
UWF_FileExclusions.txt           File/folder exclusions for all volumes
UWF_RegistryExclusions.txt       Registry exclusions
UWF_ServicingConfig.txt          UWF servicing configuration
UWF_ReadOnlyCommands.csv         Commands, exit codes and compact output
UWF_Events.csv                   UWF event records and messages
UWF_EventLog_Status.csv          Channel availability and query results
UWF_DIAGNOSTIC_SUMMARY.txt       Short UWF collection summary

Existing v1.0.4 output is retained, including WFP_State.xml, WFP_Filters.xml,
WFP_FilterInventory.csv, WFP_AllProcess_BlockCorrelation.csv,
WFP_System_Svchost_Correlation.csv, WFP_FilterOrigin_RuleDetails.csv,
WFP_Policy_Conflicts.csv, FirewallRules.csv, FirewallProfiles.csv,
ProcessInventory_All.csv, ServiceInventory_All.csv, NetworkInterfaces.csv,
AuditPolicy_ReadOnly.txt, UWF-DIAG_Report.txt, UWF-DIAG_Console.log and
RunMetadata.json.

HOW TO RUN
----------
1. Keep UWF-DIAG_v1.0.5.ps1 and Run-UWF-DIAG_v1.0.5.cmd in the same folder.
2. Double-click Run-UWF-DIAG_v1.0.5.cmd.
3. Approve the UAC prompt. The script requires elevation for complete WFP and
   Security event evidence.
4. Wait for the collection to finish. The report folder opens automatically.
5. Review UWF_DIAGNOSTIC_SUMMARY.txt first, then UWF_GetConfig.txt,
   UWF_Volumes.txt, UWF_OverlayConfig.txt, UWF_OverlayConsumption.txt and
   UWF_Events.csv.

To run with a different lookback or event limit from an elevated PowerShell:
  powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\UWF-DIAG_v1.0.5.ps1 -LookbackHours 72 -MaxUwfEvents 5000

INTERPRETING UWF STATUS
-----------------------
- Feature Enabled does not by itself prove the filter is currently active.
  Check uwfmgr get-config for current-session and next-session Filter state.
- Current and next-session states can differ because many UWF settings take
  effect only after a restart. A pending next-session state is not necessarily
  an error; compare both states and the most recent reboot time.
- Check volume get-config all to see which volumes are currently protected
  and which are configured to be protected in the next session.
- Check overlay get-config for RAM/DISK type, maximum size and thresholds.
  Check get-consumption and get-available-space for current overlay use.
- Check file and registry exclusions carefully. Excluded paths/keys bypass
  UWF filtering and can persist changes, so exclusions should be understood
  before assuming all writes are discarded on reboot.
- Check servicing get-config and the servicing-related event channels when
  assessing Windows Update or maintenance behavior.
- Event ID 1 is a warning, ID 2 is critical, and ID 3 reports return to normal
  for the uwfvol overlay threshold events documented by Microsoft.
- No event found does not prove the overlay never reached a threshold. Logs
  may be empty, disabled, cleared, inaccessible, or outside the lookback.

IMPORTANT LIMITATIONS
---------------------
- UWF availability depends on supported Windows editions/features. Microsoft
  lists Windows Enterprise, Education and IoT Enterprise as supported; Home
  and Pro are not supported for this feature.
- This diagnostic captures evidence; it does not decide whether UWF should be
  enabled or recommend changing a protected volume automatically.
- The WFP section and UWF section diagnose different subsystems: WFP handles
  network filtering; UWF controls writes to protected storage volumes.
- WFP block events do not prove UWF blocked a file write. UWF overlay events
  do not prove a network packet was blocked.
- UWF configuration and volume output format may vary by Windows version and
  feature state. Raw command output is retained so it can be examined even
  when a particular command returns a non-zero exit code.
- The collector was prepared for Windows PowerShell 5.1. It has not been run
  on the user's PC in this environment. Run it and inspect its output before
  relying on the new collection in production.

OFFICIAL MICROSOFT REFERENCES
-----------------------------
UWF WMI provider reference:
https://learn.microsoft.com/windows/configuration/unified-write-filter/uwf-wmi-provider-reference

uwfmgr command reference:
https://learn.microsoft.com/windows/configuration/unified-write-filter/uwfmgrexe

UWF overlay and thresholds:
https://learn.microsoft.com/windows/configuration/unified-write-filter/uwfoverlay

UWF troubleshooting and event logs:
https://learn.microsoft.com/windows/configuration/unified-write-filter/uwftroubleshooting

UWF servicing mode:
https://learn.microsoft.com/windows/configuration/unified-write-filter/service-uwf-protected-devices

SAFETY GUARANTEE OF THIS VERSION
--------------------------------
All UWF interactions in this diagnostic are intended to be read-only:
DISM Get-FeatureInfo, WMI class enumeration, CIM inventory queries,
uwfmgr get/get-config/get-consumption/get-exclusions commands, and event-log
queries. No UWF write/configuration command is included in the collection
sequence.
