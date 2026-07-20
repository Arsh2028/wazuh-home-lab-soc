# Wazuh Installation Guide

## Overview

This document describes the deployment of the Wazuh SIEM platform in a three-VM VirtualBox home lab.

## Lab Environment

| Component | Details |
|-----------|---------|
| Hypervisor | Oracle VirtualBox |
| Wazuh Manager | Ubuntu Server |
| Endpoint | Windows |
| Network | Internal Network |

## Installation Process

### 1. Install Ubuntu Server

- Create a new VirtualBox VM.
- Install Ubuntu Server.
- Configure a static IP address.
- Verify network connectivity.

### 2. Install Wazuh

Install the Wazuh Manager, Indexer, and Dashboard on the Ubuntu Server.

### 3. Configure the Dashboard

Access the dashboard using:

https://<server-ip>

Verify that all Wazuh services are running.

### 4. Install the Windows Agent

- Download the Wazuh agent.
- Install it on the Windows VM.
- Configure the Manager IP.
- Enroll the agent.

### 5. Verify Connectivity

Confirm that:

- The Windows agent appears in the Wazuh Dashboard.
- Security events are being collected.
- Alerts are generated successfully.

## Outcome

The Wazuh SIEM environment was successfully deployed and configured for centralized endpoint monitoring.
