# Source: https://support.originhq.com/docs/prerequisites-and-known-issues

## 

Visual C++ Redistributable

Origin agent requires **Microsoft Visual C++ 2015–2022 Redistributable (x64) 14.44**. The `.exe` installer will silently install this dependency if it is not already present on the endpoint. The `.msi` installer will need the prerequisite met or have the prerequisite package linked as part of the deployment.

## 

Network connectivity

Agent installation and ongoing operation require outbound HTTPS access to the following endpoints:

- `https://registration.prod.originhq.com`
- `https://endpoint.prod.originhq.com`

## 

SASE compatibility

> ❗️
> 
> ### 
> 
> **Important**
> 
> Compatibility considerations have been identified with several SASE traffic steering solutions. In all cases, AI-destined traffic is being routed through the SASE gateway before the Origin agent can inspect it. To ensure proper visibility, an exception must be configured to allow Origin to intercept and analyze AI-bound traffic prior to gateway enforcement.

## 

Application Control compatibility

> ❗️
> 
> ### 
> 
> **Important**
> 
> **Windows Agent:** Application Control solutions can silently block the `windivert64.sys` driver, causing AI activity to not be captured. This can be resolved by excluding `windivert64.sys` from your app control solution.

 

Copy Page