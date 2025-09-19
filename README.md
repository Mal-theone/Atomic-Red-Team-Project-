# Atomic-Red-Team-Project-
This is step by step instructions on how to Set up Atomic Red Team in your own environment 
Skip to content
Navigation Menu
Mal-theone
Atomic-Red-Team-Project-

Type / to search
Code
Issues
Pull requests
Actions
Projects
Wiki
Security
Insights
Settings
Atomic-Red-Team-Project-
/
Name your file...
in
main

Edit

Preview
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
67
68
69
70
71
72
73
74
75
76
77
78
79
80
81
82
83
84
85
86
87
88
89
90
91
92
93
94
95
96
97
98
99
100
101
102
103
104
105
106
107
108
109
110
111
112
113
114
115
116
117
118
119
120
121
122
123
124
125
126
127
128
129
130
131
132
133
134
135
136
137
138
139
140
141
142
143
144
145
146
147
148
149
150
151
152
153
154
155
156
157
158
159
160
161
162
163
164
165
166
167
168
169
170
171
172
173
174
175
176
177
178
179
180
181
182
183
184
185
186
187
188
189
190
191
192
193
194
195
196
197
198
199
200
201
202
203
204
205
206
207
208
209
210
211
212
213
214
215
216
217
218
219
220
221
222
223
224
225
226
227
228
229
230
231
232
233
234
235
236
237
238
239
240
241
242
243
244
245
246
247
248
249
250
251
252
253
254
255
256
257
258
259
260
261
262
263
264
265
266
267
268
269
270
271
272
273
274
275
276
277
278
279
280
281
282
283
284
285
286
287
288
289
290
291
292
293
294
295
296
297
298
299
300
301
302
303
304
305
306
307
308
309
310
311
312
313
314
315
316
317
318
319
320
321
322
323
324
325
326
327
328
329
330
331
332
333
334
335
336
337
338
339
340
341
342
343
344
345
346
347
348
349
350
351
352
353
354
355
356
357
358
359
360
361
362
363
364
365
366
367
368
369
370
371
372
373
374
375
376
377
378
379
380
381
382
383
384
385
# Atomic Red Team Azure Implementation Guide

## Table of Contents
1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Environment Setup](#environment-setup)
4. [Network Configuration](#network-configuration)
5. [Atomic Red Team Installation](#atomic-red-team-installation)
6. [Running Tests](#running-tests)
7. [Detection and Monitoring](#detection-and-monitoring)
8. [Troubleshooting](#troubleshooting)
9. [Security Considerations](#security-considerations)

## Overview

### What is Atomic Red Team?
Atomic Red Team is an open-source library of small, highly portable detection tests mapped to the MITRE ATT&CK framework. Maintained by Red Canary, it enables security teams to test their defenses by simulating real-world adversary tactics, techniques, and procedures (TTPs).

### Purpose of This Guide
This guide provides step-by-step instructions for deploying Atomic Red Team in Microsoft Azure, creating a controlled environment for security testing and validation.

### Architecture Overview
- **Primary Test VM**: Windows 10 machine for running Atomic Red Team tests
- **Secondary Test VM**: Additional Windows 10 machine for network-based testing scenarios
- **Network Security Groups**: Configured to allow necessary communication while maintaining security
- **Monitoring Integration**: Connected to Azure Defender and Sentinel for detection analysis

## Prerequisites

### Required Access
- Azure subscription with contributor or owner permissions
- Access to create virtual machines and network resources
- Basic understanding of PowerShell and Windows administration

### Recommended Knowledge
- Familiarity with MITRE ATT&CK framework
- Understanding of network security concepts
- Experience with Azure portal navigation

## Environment Setup

### Phase 1: Resource Group Creation

1. **Access Azure Portal**
   - Navigate to [portal.azure.com](https://portal.azure.com)
   - Sign in with your Azure credentials

2. **Create Resource Group**
   - Click "Resource groups" in the left navigation
   - Select "+ Create"
   - Configure settings:
     ```
     Subscription: [Your subscription]
     Resource group name: atomic-red-team-rg
     Region: [Select region closest to your location]
     ```
   - Click "Review + create" → "Create"

### Phase 2: Primary Virtual Machine Deployment

1. **Initiate VM Creation**
   - Click "Create a resource" or "+" icon
   - Search for "Windows 10" → Select "Microsoft Windows 10"
   - Click "Create"

2. **Basic Configuration**
   ```
   Subscription: [Your subscription]
   Resource group: atomic-red-team-rg
   Virtual machine name: atomic-primary-vm
   Region: [Same as resource group]
   Availability options: No infrastructure redundancy required
   Security type: Standard
   Image: Windows 10 Pro, Version 22H2 - x64 Gen2
   Size: Standard_B2s (2 vcpus, 4 GiB memory)
   ```

3. **Administrator Account**
   ```
   Username: azureuser
   Password: [Create strong password - save securely]
   Confirm password: [Repeat password]
   ```

4. **Inbound Port Rules**
   - Select "Allow selected ports"
   - Select "RDP (3389)"

5. **Licensing**
   - Check "I confirm I have an eligible Windows 10/11 license"

6. **Complete Creation**
   - Click "Review + create" → "Create"
   - Wait 5-10 minutes for deployment

### Phase 3: Secondary Virtual Machine Deployment

1. **Create Second VM**
   - Follow same process as Primary VM with these changes:
     ```
     Virtual machine name: atomic-secondary-vm
     ```

2. **Critical Networking Step**
   - Navigate to "Networking" tab before creating
   - Verify settings:
     ```
     Virtual network: [Same as primary VM]
     Subnet: [Same as primary VM]
     Public IP: Create new
     NIC network security group: Basic
     Public inbound ports: Allow RDP (3389)
     ```

3. **Deploy Second VM**
   - Click "Review + create" → "Create"

## Network Configuration

### Understanding Network Security Groups (NSGs)

NSGs act as virtual firewalls controlling traffic flow. Rules are processed by priority (lower numbers = higher priority).

### RDP Access Configuration

Both VMs should have RDP access configured automatically. To verify or modify:

1. **Navigate to VM Networking**
   - Go to Virtual Machine → Networking → Inbound port rules
   
2. **Verify RDP Rule Exists**
   ```
   Priority: 1000
   Name: RDP
   Port: 3389
   Protocol: TCP
   Source: Any
   Action: Allow
   ```

### ICMP Communication Setup

To enable ping testing between VMs:

1. **Configure Secondary VM (Inbound ICMP)**
   - Go to atomic-secondary-vm → Networking
   - Click "Add inbound port rule"
   - Configure:
     ```
     Source: Any
     Source port ranges: *
     Destination: Any
     Service: Custom
     Destination port ranges: *
     Protocol: ICMP
     Action: Allow
     Priority: 310
     Name: Allow-ICMP-Inbound
     ```

2. **Configure Primary VM (Outbound ICMP)**
   - Go to atomic-primary-vm → Networking → Outbound port rules
   - Click "Add outbound port rule"
   - Configure:
     ```
     Source: Any
     Source port ranges: *
     Destination: Any
     Service: Custom
     Destination port ranges: *
     Protocol: ICMP
     Action: Allow
     Priority: 310
     Name: Allow-ICMP-Outbound
     ```

3. **Configure Windows Firewall**
   - RDP into both VMs
   - Run PowerShell as Administrator
   - Execute on both machines:
     ```powershell
     New-NetFirewallRule -DisplayName "Allow ICMPv4-In" -Direction Inbound -Protocol ICMPv4 -Action Allow
     New-NetFirewallRule -DisplayName "Allow ICMPv4-Out" -Direction Outbound -Protocol ICMPv4 -Action Allow
     ```

### Connectivity Verification

Test connectivity between VMs:
```powershell
# From primary VM, ping secondary VM
ping [secondary-vm-private-ip]

# Test should return successful responses
```

## Atomic Red Team Installation

### Phase 1: Prerequisites Installation

1. **Connect to Primary VM**
   - Use RDP with VM's public IP and credentials
   - Open PowerShell as Administrator

2. **Install Git**
   - Download Git for Windows from [git-scm.com](https://git-scm.com)
   - Install with default settings
   - Restart PowerShell after installation

3. **Verify Git Installation**
   ```powershell
   git --version
   ```

### Phase 2: Repository Setup

1. **Create Working Directory**
   ```powershell
   New-Item -Path "C:\AtomicRedTeam" -ItemType Directory
   Set-Location "C:\AtomicRedTeam"
   ```

2. **Clone Repository**
   ```powershell
   git clone https://github.com/redcanaryco/atomic-red-team.git
   ```

3. **Verify Download**
   ```powershell
   Get-ChildItem "C:\AtomicRedTeam\atomic-red-team"
   ```

### Phase 3: PowerShell Module Installation

1. **Set Execution Policy**
   ```powershell
   Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Force
   ```

2. **Install Atomic Red Team Module**
   ```powershell
   Install-Module -Name invoke-atomicredteam -Scope CurrentUser -Force
   ```

3. **Import Module**
   ```powershell
   Import-Module invoke-atomicredteam -Force
   ```

4. **Verify Installation**
   ```powershell
   Get-Command -Module invoke-atomicredteam
   ```

## Running Tests

### Understanding Test Structure

Atomic tests are identified by MITRE ATT&CK technique IDs (e.g., T1082 for System Information Discovery).

### Your First Test

1. **List Available Tests**
   ```powershell
   Invoke-AtomicTest T1082 -ShowDetailsBrief
   ```

2. **Run Test with Prerequisites Check**
   ```powershell
   Invoke-AtomicTest T1082 -CheckPrereqs -PathToAtomicsFolder "C:\AtomicRedTeam\atomic-red-team\atomics"
   ```

3. **Execute the Test**
   ```powershell
   Invoke-AtomicTest T1082 -PathToAtomicsFolder "C:\AtomicRedTeam\atomic-red-team\atomics"
   ```

### Advanced Test Execution

1. **Run Specific Test Number**
   ```powershell
   Invoke-AtomicTest T1082-1 -PathToAtomicsFolder "C:\AtomicRedTeam\atomic-red-team\atomics"
   ```

2. **Cleanup After Test**
   ```powershell
   Invoke-AtomicTest T1082 -Cleanup -PathToAtomicsFolder "C:\AtomicRedTeam\atomic-red-team\atomics"
   ```

3. **Run All Tests for a Technique**
   ```powershell
   Invoke-AtomicTest T1082 -PathToAtomicsFolder "C:\AtomicRedTeam\atomic-red-team\atomics" -ExecutionLogPath "C:\AtomicRedTeam\logs\execution.log"
   ```

## Detection and Monitoring

### Azure Integration Options

For comprehensive detection and analysis, integrate with Azure security services:

#### Microsoft Defender for Endpoint
- Provides real-time detection and response capabilities
- Automatically correlates Atomic Red Team activities with threat intelligence

#### Microsoft Sentinel
- Centralized SIEM for log aggregation and analysis
- Custom detection rules for Atomic Red Team techniques

### Log Collection Points

Monitor these locations for test artifacts:
- Windows Event Logs (Security, System, Application)
- PowerShell logs
- Process creation events
- Network connection logs
- File system changes

## Troubleshooting

### Common Issues and Solutions

1. **PowerShell Execution Policy Error**
   ```powershell
   Set-ExecutionPolicy -ExecutionPolicy Bypass -Scope Process -Force
   ```

2. **Module Installation Fails**
   ```powershell
   Install-PackageProvider -Name NuGet -MinimumVersion 2.8.5.201 -Force
   Set-PSRepository -Name PSGallery -InstallationPolicy Trusted
   ```

3. **Git Clone Fails**
   - Verify internet connectivity
   - Check firewall settings
   - Try using HTTPS instead of SSH

4. **ICMP Tests Don't Work**
   - Verify NSG rules are correctly configured
   - Check Windows Firewall settings on both VMs
   - Confirm VMs are in the same virtual network

### Diagnostic Commands

```powershell
# Check module status
Get-Module invoke-atomicredteam -ListAvailable

# Verify network connectivity
Test-NetConnection -ComputerName [target-ip] -Port 3389

# Check execution policy
Get-ExecutionPolicy -List
```

## Security Considerations

### Environment Isolation
- Use dedicated resource groups for testing
- Implement network segmentation
- Consider using Azure Bastion for secure RDP access

### Access Control
- Limit VM access to authorized personnel only
- Use strong passwords and consider MFA
- Implement Just-In-Time VM access

### Monitoring and Alerting
- Enable Azure Security Center
- Configure log forwarding to Sentinel
- Set up alerts for suspicious activities

### Cleanup Procedures
- Run cleanup commands after each test
- Monitor for persistent changes
- Document all testing activities

### Best Practices
- Test in non-production environments only
- Maintain current backups
- Review security logs regularly
- Keep Atomic Red Team repository updated

---

**Note**: This testing environment is designed for security research and detection validation. Always ensure proper authorization before conducting any security testing activities.
New File at / · Mal-theone/Atomic-Red-Team-Project-
