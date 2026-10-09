Companies use several operating systems for servers, depending on their applications, infrastructure, budget, security requirements, and existing IT environment.

For someone learning Linux server administration, Linux distributions—especially Ubuntu Server and Red Hat Enterprise Linux (RHEL)—are excellent starting points. Windows Server is also widely used in enterprise environments, particularly for Microsoft-based infrastructure.

## Common server operating systems used in companies

[Beginners’ Guide To Run A Linux Server Securely - DEV Community](https://images.openai.com/static-rsc-4/G0E9OZz8lt-B7HCAVAoYMbN_6pPryujD6J6mGDtGml-N6_4pQO3bAM4vrIe4vFwTB4zcMJGSQoYH8-_DzjyW3Zh5Qm2mUp0Thzr7nioUlbLAi4h_nDGphV2-QwSn-aTJRh8yJGmAaUWfXWdBkA9o7A6mmK3lk0lHXZTuTSr2w0I?purpose=inline)

1\. Ubuntu Server

Great starting point

Used for web servers, cloud workloads, application hosting, DevOps, containers and automation. Beginner-friendly and supported by extensive documentation.

[Monitoring System Memory Usage with Shell Script | by Shravanth Gelli | Medium](https://images.openai.com/static-rsc-4/60GaDhhE35GURg7VMwlrFmW-DKxlU61CyrAB5FvLfm7movt1a2dzdFShG1P8g6qOwb9zj5zz0MXW9ltMVZsp7paWnKTOcH_JtrDSeT10Ur7LtdMYphjYLQqHBT0gPZmi_C-b7Mb6Byw0kbwiyG0JYLsPadUVQpnC8G-81Q08rIo?purpose=inline)

2\. Red Hat Enterprise Linux (RHEL)

Common in enterprise IT, regulated environments, enterprise applications and organizations requiring commercial support. Related distributions include Rocky Linux and AlmaLinux.

[Windows Server 2019 : Active Directory : Install : Server World](https://images.openai.com/static-rsc-4/VwAL59VpU3JzMNPHDqDo2XYsajRnz5r1KzPWWd6MmPmEQVZn4sgy0oc5VAxzrP_TRql_0Pzni1O2BBbqLv7JFx0eHD0vSTuv9HDL33h23qxNsJR01bD20r4ayBzUk1CLu1MyB2naHg4mWBEslZOlMOzw5XxYNFFwqtNWiJFIz8U?purpose=inline)

3\. Windows Server

Used for Active Directory, Group Policy, Windows-based applications, file and print services, and Microsoft infrastructure.

[Changing the Resolution and Font Type in Debian Server](https://images.openai.com/static-rsc-4/f_uLmLhAp1lS20xANGeMfUilJ_g6ge4v4PTH7584MLs_cs50oBZN3xxhHNJCYjqm3LRshxCwe1tcDcp4AOPzfEsFFILgE0wjrbGm6uwYCckhNzlJlgdeUypVWjq8z5Sn8_2iufG-n7uBZurv0WOExHDiaijPAQCiu8fXYR7Te9M?purpose=inline)

4\. Debian

Valued for stability and used for web hosting, infrastructure services and custom server deployments.

[Conexión a instancias de Linux o Unix en Lightsail - Amazon Lightsail](https://images.openai.com/static-rsc-4/dJnoMzXQc0YXxmCHBynqywCA4m_4ICU7DEFFaZ7jos2y17kxKSD5QszP7CF83pybGRxSEE9vO1_gVGYAShPQj2mJsukchU4GsplqXLdstPsdeQ1CNFBTthrOfTym2lYxYP6PQvnLRVzZO_twsVXJebAKmFPLWUCWCS4yzqaarqs?purpose=inline)

5\. Amazon Linux

Designed for AWS workloads, including cloud applications, automation and services running on Amazon EC2.

## What should you learn for a job?

| Career path                            | Recommended OS to learn                                |
| -------------------------------------- | ------------------------------------------------------ |
| Linux System Administrator             | Ubuntu Server, then RHEL                               |
| DevOps Engineer                        | Ubuntu/RHEL, then Docker and Kubernetes                |
| Cloud Engineer                         | Linux, then AWS/Azure                                  |
| Windows System Administrator           | Windows Server, Active Directory, PowerShell           |
| Network / Infrastructure Administrator | Linux basics, Windows Server, networking               |
| Cybersecurity / SOC                    | Linux, Windows administration, logs and security tools |

## A practical learning roadmap

Since you're already running Ubuntu Server in a virtual machine, I'd follow this sequence:

Stage 1 — Ubuntu Server

Linux commands, users, permissions, SSH, networking, systemd and logs.

Stage 2 — Server services

Nginx, DNS concepts, DHCP, firewalls, storage and backups.

Stage 3 — Enterprise skills

RHEL fundamentals, Docker, Bash scripting, Git and automation.

Stage 4 — Cloud infrastructure

AWS or Azure, virtual machines, networking, monitoring and deployment.

My advice: Install Ubuntu Server LTS in your VM and learn it thoroughly first. Don't switch operating systems just because companies use different distributions. The core Linux administration skills transfer between them.

One useful distinction: companies may run Linux or Windows inside virtual machines, while their physical servers run a hypervisor such as VMware ESXi or Hyper-V. Cloud providers also manage much of the underlying hardware for you.
