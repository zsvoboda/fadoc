# Creating a File Virtual Interface
FlashArray File Services uses a virtual interface to communicate with client computers for both resiliency and performance. The File Service virtual interface (also known as a file VIF) consists of four or more physical Ethernet ports across both array controllers. This design protects the array from individual port or network link failures, as well as from controller failures.

FlashArray supports two different file virtual interface configurations:
- **Physical Bonding**: The file virtual interface uses pairs of the same physical ports on both controllers. For example, `ct0.eth4` and `ct1.eth4`, or `ct0.eth5` and `ct1.eth5`.
- **LACP (Link Aggregation Control Protocol) Bonding**: The file virtual interface uses a pair of LACP virtual interfaces, each on a different controller. An LACP virtual interface can use up to four physical Ethernet ports on the same controller. For example, `ct0.eth4` and `ct0.eth5`.
> **Note**: Your network infrastructure (switches and routers) must support the LACP standard (IEEE 802.3ad).

The LACP configuration is preferred over physical bonding.
## Creating a File Virtual Interface: Physical Bonding
To create a file virtual interface using physical bonding, you need at least one pair of physical Ethernet ports. The pair must consist of the same physical Ethernet ports on different controllers. For example, you can select the `ct0.eth4` and `ct1.eth4` physical ports.

The recommended configuration is to use two pairs of physical Ethernet ports in the file virtual interface setup. Avoid using the `management` or `replication` physical Ethernet ports for file virtual interface creation. ### FlashArray GUI
First, identify the pairs of physical Ethernet ports to be used for the new file virtual interface. Ensure that the selected physical Ethernet ports are not already in use by management services, a bond, or subnet configuration. The selected ports must also have the same MTU value.

Navigate to the `Settings/NETWORK/Connectors` configuration page to review the available physical Ethernet ports.

![Settings/NETWORK/Connectors](https://github.com/zsvoboda/fadoc/blob/main/src/img/vif.physical/settings.network.configuration.png)

In this example, we will use the `eth4` and `eth5` ports. Right-click on the Ethernet widget's menu and select the `Create Virtual Network Interface` option.

![Create Virtual Network Interface](https://github.com/zsvoboda/fadoc/blob/main/src/img/vif.physical/create.file.vif.png)

In the `Create Virtual Network Interface` dialog, enter the `Name` and `IP Address`, then click the edit icon next to the `Subinterfaces` field.

![Create Virtual Network Interface Dialog](https://github.com/zsvoboda/fadoc/blob/main/src/img/vif.physical/create.file.vif.dialog.png)

Select the `eth4` and `eth5` physical Ethernet port pairs.

![Select Subinterfaces](https://github.com/zsvoboda/fadoc/blob/main/src/img/vif.physical/create.file.vif.dialog.subinterfaces.png)

Click the `Select` button in the dialog, then click the `Add` button in the previous dialog.

#### Edit and Enable the File Virtual Interface using GUI
Find the new `filevif0` file virtual interface in the `Settings/NETWORK/Connectors` page and click the `Edit` button.

![Edit Virtual Network Interface](https://github.com/zsvoboda/fadoc/blob/main/src/img/vif.physical/edit.file.vif.png)

Specify the `Netmask` and `Gateway` values, and enable the new file virtual interface.

![Edit Virtual Network Interface Dialog](https://github.com/zsvoboda/fadoc/blob/main/src/img/vif.physical/edit.file.vif.dialog.png)

If you no longer need the file virtual interface, you can delete it by clicking the `Delete` command in the Ethernet list.

![Delete Virtual Network Interface](https://github.com/zsvoboda/fadoc/blob/main/src/img/vif.physical/delete.file.vif.png)


### FlashArray Command Line Interface
First, identify the pairs of physical Ethernet ports to be used for the new file virtual interfaces. Ensure that the physical Ethernet ports you select are not already in use by management services, in a bond, or for subnet configuration. The selected ports must also have the same MTU value.

You can use the following command to list all existing network interfaces on the array:

```bash
myarray-ct0:~# purenetwork eth list
Name      Enabled  Type      Subnet  Address        Mask           Gateway      MTU   MAC                Speed       Services     Subinterfaces
ct0.eth0  True     physical  -       10.7.88.156    255.255.248.0  10.7.88.1    9000  00:50:56:84:68:ac  10.00 Gb/s  management   -
ct0.eth1  True     physical  -       10.7.88.160    255.255.248.0  10.7.88.1    9000  00:50:56:84:69:c6  10.00 Gb/s  management   -
ct0.eth2  True     physical  -       -              -              -            9000  00:50:56:84:bb:e9  10.00 Gb/s  replication  -
ct0.eth3  True     physical  -       -              -              -            9000  00:50:56:84:cb:96  10.00 Gb/s  replication  -
ct0.eth4  True     physical  -       -              -              -            9000  00:50:56:84:55:f8  10.00 Gb/s  iscsi        -
ct0.eth5  True     physical  -       -              -              -            9000  00:50:56:84:77:1a  10.00 Gb/s  iscsi        -
ct1.eth0  True     physical  -       10.7.88.163    255.255.248.0  10.7.88.1    9000  00:50:56:84:a6:42  10.00 Gb/s  management   -
ct1.eth1  True     physical  -       10.7.88.165    255.255.248.0  10.7.88.1    9000  00:50:56:84:93:53  10.00 Gb/s  management   -
ct1.eth2  True     physical  -       -              -              -            9000  00:50:56:84:bd:9b  10.00 Gb/s  replication  -
ct1.eth3  True     physical  -       -              -              -            9000  00:50:56:84:ec:65  10.00 Gb/s  replication  -
ct1.eth4  True     physical  -       -              -              -            9000  00:50:56:84:5e:7b  10.00 Gb/s  iscsi        -
ct1.eth5  True     physical  -       -              -              -            9000  00:50:56:84:60:fd  10.00 Gb/s  iscsi        -
replbond  False    bond      -       -              -              -            1500  9e:df:a2:8e:da:19  0.00 b/s    replication  -
vir0      True     vif       -       10.7.88.167    255.255.248.0  10.7.88.1    9000  02:40:e5:ea:68:b3  10.00 Gb/s  management   -
vir1      False    vif       -       -              -              -            9000  d6:4a:35:5b:d7:e9  10.00 Gb/s  management   -
```

In this example, we’ll select the `eth4` and `eth5` interfaces. First, create the file virtual interface using the following command:

```bash
purenetwork eth create vif --address 192.168.1.231 --subinterfacelist ct0.eth4,ct1.eth4,ct0.eth5,ct1.eth5 --servicelist file filevif0
```
#### Edit and Enable the File Virtual Interface using CLI
Next, configure the netmask and gateway for the new interface:

```bash
purenetwork eth setattr --netmask 255.255.255.0 --gateway 192.168.1.1 filevif0
```

Finally, enable the new interface:

```bash
purenetwork eth enable filevif0
```

You can double-check that the file virtual interface has been successfully created by listing its details:

```bash
purenetwork eth list filevif0
Name      Enabled  Type  Subnet  Address        Mask           Gateway      MTU   MAC                Speed       Services  Subinterfaces
filevif0  True     vif   -       192.168.1.231  255.255.255.0  192.168.1.1  9000  b6:de:eb:d9:39:17  10.00 Gb/s  file      ct0.eth4
                                                                                                                           ct1.eth4
                                                                                                                           ct0.eth5
                                                                                                                           ct1.eth5
```

If you no longer need the file virtual interface, you can delete it with the following command:

```bash
purenetwork eth delete filevif0
```
## Creating a File Virtual Interface: LACP

Link Aggregation Control Protocol (LACP) is an IEEE standard that allows individual Ethernet links to be aggregated into a single logical Ethernet link. Depending on a on the scenario, it can be used to increase bandwidth utilization, increase availability, or simplify network configurations. In order for LACP to work with the FlashArray, the network switch must be configured for LACP as well.

To create a file virtual interface using LACP bonding, you need at least two physical Ethernet ports. Each pair must consist of different Ethernet ports on the same controller. For example, you can select the `ct0.eth4` and `ct0.eth5` physical Ethernet ports.

The recommended configuration is to use two LACP bonds, each with two physical Ethernet ports. Avoid using the `management` or `replication` physical Ethernet ports for LACP bond creation.### FlashArray GUI
First, we need to create two LACP virtual interfaces. Begin by identifying the groups of physical Ethernet ports to be used for the new LACP virtual interfaces. All selected ports must reside on the same controller. Ensure that these physical Ethernet ports are not already in use by management services, another bond, or subnet configuration. Additionally, the selected ports must have the same MTU value.

Navigate to the `Settings/NETWORK/Connectors` configuration page to review the available physical Ethernet ports.

![Settings/NETWORK/Connectors](https://github.com/zsvoboda/fadoc/blob/main/src/img/vif.physical/settings.network.configuration.png)

In this example, we will use the `ct0.eth4` and `ct0.eth5` ports. Right-click on the Ethernet widget's menu and select the `Create LACP Bond` option.

![Create LACP Bond](https://github.com/zsvoboda/fadoc/blob/main/src/img/vif.lacp/create.lacp.png)

In the `Create LACP Bond` dialog, enter the `Name`, then click the edit icon next to the `Subinterfaces` field.

![Create LACP Dialog](https://github.com/zsvoboda/fadoc/blob/main/src/img/vif.lacp/create.lacp.dialog.0.png)

Select the `ct0.eth4` and `ct0.eth5` physical Ethernet ports.

![Select Subinterfaces](https://github.com/zsvoboda/fadoc/blob/main/src/img/vif.lacp/create.lacp.dialog.subinterfaces.0.png)

Click the `Select` button in the dialog, then click the `Add` button in the previous dialog.

Repeat the process for the `ct1.eth4` and `ct1.eth5` physical Ethernet ports to create the `lacp1` LACP bond.

![Create LACP Dialog](https://github.com/zsvoboda/fadoc/blob/main/src/img/vif.lacp/create.lacp.dialog.1.png)
![Select Subinterfaces](https://github.com/zsvoboda/fadoc/blob/main/src/img/vif.lacp/create.lacp.dialog.subinterfaces.1.png)

Navigate to the `Settings/NETWORK/Connectors` configuration page to enable the newly created LACP bonds.

![Settings/NETWORK/Connectors](https://github.com/zsvoboda/fadoc/blob/main/src/img/vif.lacp/settings.network.configuration.png)

Enable both `lacp0` and `lacp1` LACP bonds. You don’t need to fill in any other values; simply set the `Enabled` radio button.

Enable `lacp0`:

![Enable LACP Bond](https://github.com/zsvoboda/fadoc/blob/main/src/img/vif.lacp/enable.lacp.bond.0.png)

and `lacp1`:

![Enable LACP Bond](https://github.com/zsvoboda/fadoc/blob/main/src/img/vif.lacp/enable.lacp.bond.1.png)

Now, we’ll create a new file virtual interface using the LACP bonds.

![Create Virtual Network Interface](https://github.com/zsvoboda/fadoc/blob/main/src/img/vif.physical/create.file.vif.png)

In the `Create Virtual Network Interface` dialog, enter the `Name` and `IP Address`, then click the edit icon next to the `Subinterfaces` field.

![Create Virtual Network Interface Dialog](https://github.com/zsvoboda/fadoc/blob/main/src/img/vif.physical/create.file.vif.dialog.png)

Select the `lacp0` and `lacp1` LACP bonds.

![Select Subinterfaces](https://github.com/zsvoboda/fadoc/blob/main/src/img/vif.lacp/create.file.vif.dialog.subinterfaces.png)

Click the `Select` button in the dialog, then click the `Add` button in the previous dialog.
#### Edit and Enable the File Virtual Interface using GUI
Find the new `filevif0` file virtual interface in the `Settings/NETWORK/Connectors` page and click the `Edit` button.

![Edit Virtual Network Interface](https://github.com/zsvoboda/fadoc/blob/main/src/img/vif.lacp/edit.file.vif.png)

Specify the `Netmask` and `Gateway` values, and enable the new file virtual interface.

![Edit Virtual Network Interface Dialog](https://github.com/zsvoboda/fadoc/blob/main/src/img/vif.physical/edit.file.vif.dialog.png)

If you no longer need the file virtual interface, you can delete it by clicking the `Delete` command in the Ethernet list.

![Delete Virtual Network Interface](https://github.com/zsvoboda/fadoc/blob/main/src/img/vif.physical/delete.file.vif.png)



### FlashArray Command Line Interface
First, we need to create two LACP virtual interfaces. Begin by identifying the groups of physical Ethernet ports to be used for the new LACP virtual interfaces. All selected ports must reside on the same controller. Ensure that these physical Ethernet ports are not already in use by management services, another bond, or subnet configuration. Additionally, the selected ports must have the same MTU value.

You can use the following command to list all existing network interfaces on the array:

```bash
myarray-ct0:~# purenetwork eth list
Name      Enabled  Type      Subnet  Address        Mask           Gateway      MTU   MAC                Speed       Services     Subinterfaces
ct0.eth0  True     physical  -       10.7.88.156    255.255.248.0  10.7.88.1    9000  00:50:56:84:68:ac  10.00 Gb/s  management   -
ct0.eth1  True     physical  -       10.7.88.160    255.255.248.0  10.7.88.1    9000  00:50:56:84:69:c6  10.00 Gb/s  management   -
ct0.eth2  True     physical  -       -              -              -            9000  00:50:56:84:bb:e9  10.00 Gb/s  replication  -
ct0.eth3  True     physical  -       -              -              -            9000  00:50:56:84:cb:96  10.00 Gb/s  replication  -
ct0.eth4  True     physical  -       -              -              -            9000  00:50:56:84:55:f8  10.00 Gb/s  iscsi        -
ct0.eth5  True     physical  -       -              -              -            9000  00:50:56:84:77:1a  10.00 Gb/s  iscsi        -
ct1.eth0  True     physical  -       10.7.88.163    255.255.248.0  10.7.88.1    9000  00:50:56:84:a6:42  10.00 Gb/s  management   -
ct1.eth1  True     physical  -       10.7.88.165    255.255.248.0  10.7.88.1    9000  00:50:56:84:93:53  10.00 Gb/s  management   -
ct1.eth2  True     physical  -       -              -              -            9000  00:50:56:84:bd:9b  10.00 Gb/s  replication  -
ct1.eth3  True     physical  -       -              -              -            9000  00:50:56:84:ec:65  10.00 Gb/s  replication  -
ct1.eth4  True     physical  -       -              -              -            9000  00:50:56:84:5e:7b  10.00 Gb/s  iscsi        -
ct1.eth5  True     physical  -       -              -              -            9000  00:50:56:84:60:fd  10.00 Gb/s  iscsi        -
replbond  False    bond      -       -              -              -            1500  9e:df:a2:8e:da:19  0.00 b/s    replication  -
vir0      True     vif       -       10.7.88.167    255.255.248.0  10.7.88.1    9000  02:40:e5:ea:68:b3  10.00 Gb/s  management   -
vir1      False    vif       -       -              -              -            9000  d6:4a:35:5b:d7:e9  10.00 Gb/s  management   -
```

In this example, we’ll use the `ct0.eth4` and `ct0.eth5` interfaces for creation of the LACP bond for the controller `ct0`. 

```bash
purenetwork eth create lacpbond --subinterfacelist ct0.eth4,ct0.eth5 lacp0
```

Then we'll use the `ct1.eth4` and `ct1.eth5` interfaces for creation of the LACP bond for the controller `ct1`.

```bash
purenetwork eth create lacpbond --subinterfacelist ct1.eth4,ct1.eth5 lacp1
```

Then we'll enable both LACP bonds `lacp0` and `lacp1`

```bash
purenetwork eth enable lacp0
purenetwork eth enable lacp1
```

Then we create a new file virtual interface using these LACP bonds

```bash
purenetwork eth create vif --address 192.168.1.231 --subinterfacelist lacp0,lacp1 --servicelist file filevif0
```


#### Edit and Enable the File Virtual Interface using CLI
Next, configure the netmask and gateway for the new interface:

```bash
purenetwork eth setattr --netmask 255.255.255.0 --gateway 192.168.1.1 filevif0
```

Finally, enable the new interface:

```bash
purenetwork eth enable filevif0
```

You can double-check that the file virtual interface has been successfully created by listing its details:

```bash
purenetwork eth list filevif0
Name      Enabled  Type  Subnet  Address        Mask           Gateway      MTU   MAC                Speed       Services  Subinterfaces
filevif0  True     vif   -       192.168.1.231  255.255.255.0  192.168.1.1  9000  3a:98:82:5a:f6:1c  20.00 Gb/s  file      lacp0
                                                                                                                           lacp1
```

If you no longer need the LACP bonds and file virtual interface, you can delete it with the following command:

```bash
purenetwork eth delete filevif0
purenetwork eth delete lacp0
purenetwork eth delete lacp1
```
# Creating a File Virtual Interface using VLAN in the GUI

Virtual LAN (VLAN) tagging is a networking technology that allows for the logical division of a network into multiple virtual networks, called VLANs, to improve network security, performance, and flexibility.

In FlashArray, VLANs are represented by subnets. You can create a new subnet from the `Subnets` widget on the `Settings/NETWORK/Configuration` page:

![Subnets List](https://github.com/zsvoboda/fadoc/blob/main/src/img/vlan/subnets.list.png)

Click on the `+` button to add a new subnet.

![Create Subnet](https://github.com/zsvoboda/fadoc/blob/main/src/img/vlan/create.subnet.png)

Once the VLAN subnet is created, you need to add physical Ethernet interfaces to the subnet by clicking the `Add interface +` button:

![Add Interface](https://github.com/zsvoboda/fadoc/blob/main/src/img/vlan/add.interface.png)

This will open the following dialog:

![Interface ct0.eth4](https://github.com/zsvoboda/fadoc/blob/main/src/img/vlan/add.interface.ct0.eth4.png)

Here, you don't need to specify the last component of the IP address for the interface.

You will need to repeat this process for all interfaces you plan to use for the file virtual interface. In our example, these are `ct0.eth4`, `ct1.eth4`, `ct0.eth5`, and `ct1.eth5`.

![Interface ct1.eth4](https://github.com/zsvoboda/fadoc/blob/main/src/img/vlan/add.interface.ct1.eth4.png)
![Interface ct0.eth5](https://github.com/zsvoboda/fadoc/blob/main/src/img/vlan/add.interface.ct0.eth5.png)
![Interface ct1.eth5](https://github.com/zsvoboda/fadoc/blob/main/src/img/vlan/add.interface.ct1.eth5.png)

You can now see the new VLAN virtual interfaces in the `Ethernet` widget on the `Settings/NETWORK/Connectors` page. These virtual interfaces are ready to be used for creating a new file virtual interface.

![VLAN Interfaces List](https://github.com/zsvoboda/fadoc/blob/main/src/img/vlan/vlan.interfaces.list.png)

To create the virtual network interface, click on the `Create Virtual Network Interface` menu item and fill in the details for the new file virtual network interface.

![File Virtual Interface Dialog](https://github.com/zsvoboda/fadoc/blob/main/src/img/vlan/vlan.vif.dialog.png)

Next, select the VLAN virtual interfaces in the `Select Subinterfaces` dialog:

![Subinterfaces Dialog](https://github.com/zsvoboda/fadoc/blob/main/src/img/vlan/vlan.vif.dialog.subinterfaces.png)

Click the `Select` button in the dialog, then click the `Add` button in the previous dialog.

Find the new `filevif0` file virtual interface in the `Settings/NETWORK/Connectors` page and click the `Edit` button.

![Edit Virtual Network Interface](https://github.com/zsvoboda/fadoc/blob/main/src/img/vlan/edit.vlan.vif.png)

Specify the `Netmask` and `Gateway` values, and enable the new file virtual interface.

![Edit Virtual Network Interface Dialog](https://github.com/zsvoboda/fadoc/blob/main/src/img/vif.physical/edit.file.vif.dialog.png)

If you no longer need the file virtual interface, you can delete it by clicking the `Delete` command in the Ethernet list.

![Delete File Virtual Network Interface](https://github.com/zsvoboda/fadoc/blob/main/src/img/vif.physical/delete.file.vif.png)

You can also delete the VLAN virtual network interfaces by clicking the `x` button next to a specific interface in the `Subnets` widget on the `Settings/NETWORK/Configuration` page.

![Delete VLAN Virtual Network Interface](https://github.com/zsvoboda/fadoc/blob/main/src/img/vlan/delete.vlan.vifs.png)

Finally, you can also delete the VLAN subnet by clicking on the `delete` icon in the `Subnets` widget on the `Settings/NETWORK/Configuration` page  if it is no longer needed.

![Delete Subnet](https://github.com/zsvoboda/fadoc/blob/main/src/img/vlan/delete.subnet.png)

# Creating a File Virtual Interface using LACP with VLAN in the GUI

Alternatively, you can create a new file virtual interface from LACP bonds in VLAN subnet:

In this example, we will use the `ct0.eth4` and `ct0.eth5` ports. Right-click on the Ethernet widget's menu and select the `Create LACP Bond` option.

![Create LACP Bond](https://github.com/zsvoboda/fadoc/blob/main/src/img/vif.lacp/create.lacp.png)

In the `Create LACP Bond` dialog, enter the `Name`, then click the edit icon next to the `Subinterfaces` field.

![Create LACP Dialog](https://github.com/zsvoboda/fadoc/blob/main/src/img/vif.lacp/create.lacp.dialog.0.png)

Select the `ct0.eth4` and `ct0.eth5` physical Ethernet ports.

![Select Subinterfaces](https://github.com/zsvoboda/fadoc/blob/main/src/img/vif.lacp/create.lacp.dialog.subinterfaces.0.png)

Click the `Select` button in the dialog, then click the `Add` button in the previous dialog.

Repeat the process for the `ct1.eth4` and `ct1.eth5` physical Ethernet ports to create the `lacp1` LACP bond.

![Create LACP Dialog](https://github.com/zsvoboda/fadoc/blob/main/src/img/vif.lacp/create.lacp.dialog.1.png)
![Select Subinterfaces](https://github.com/zsvoboda/fadoc/blob/main/src/img/vif.lacp/create.lacp.dialog.subinterfaces.1.png)

Navigate to the `Settings/NETWORK/Connectors` configuration page to enable the newly created LACP bonds.

![Settings/NETWORK/Connectors](https://github.com/zsvoboda/fadoc/blob/main/src/img/vif.lacp/settings.network.configuration.png)

Enable both `lacp0` and `lacp1` LACP bonds. You don’t need to fill in any other values; simply set the `Enabled` radio button.

Enable `lacp0`:

![Enable LACP Bond](https://github.com/zsvoboda/fadoc/blob/main/src/img/vif.lacp/enable.lacp.bond.0.png)

and `lacp1`:

![Enable LACP Bond](https://github.com/zsvoboda/fadoc/blob/main/src/img/vif.lacp/enable.lacp.bond.1.png)

Then create a new subnet from the `Subnets` widget on the `Settings/NETWORK/Configuration` page:

![Subnets List](https://github.com/zsvoboda/fadoc/blob/main/src/img/vlan/subnets.list.png)

Click on the `+` button to add a new subnet.

![Create Subnet](https://github.com/zsvoboda/fadoc/blob/main/src/img/vlan/create.subnet.png)

Once the VLAN subnet is created, you need to add the LACP bonds to the subnet by clicking the `Add interface +` button:

![Add Interface](https://github.com/zsvoboda/fadoc/blob/main/src/img/vlan/add.interface.png)

This will open the following dialog:

![Interface ct0.eth4](https://github.com/zsvoboda/fadoc/blob/main/src/img/vlan/add.interface.lacp0.png)

Here, you don't need to specify the last component of the IP address for the interface.

You will need to repeat this process for the `lacp1`.

![Interface ct1.eth4](https://github.com/zsvoboda/fadoc/blob/main/src/img/vlan/add.interface.lacp1.png)

You can now see the new VLAN LACP bonds in the `Ethernet` widget on the `Settings/NETWORK/Connectors` page. These VLAN LACP bonds are ready to be used for creating a new file virtual interface.

![VLAN Interfaces List](https://github.com/zsvoboda/fadoc/blob/main/src/img/vlan/vlan.lacp.interfaces.list.png)

To create the virtual network interface, click on the `Create Virtual Network Interface` menu item and fill in the details for the new file virtual network interface.

![File Virtual Interface Dialog](https://github.com/zsvoboda/fadoc/blob/main/src/img/vlan/vlan.vif.dialog.png)

Next, select the VLAN virtual interfaces in the `Select Subinterfaces` dialog:

![Subinterfaces Dialog](https://github.com/zsvoboda/fadoc/blob/main/src/img/vlan/vlan.vif.dialog.lacp.subinterfaces.png)

Click the `Select` button in the dialog, then click the `Add` button in the previous dialog.

Find the new `filevif0` file virtual interface in the `Settings/NETWORK/Connectors` page and click the `Edit` button.

![Edit Virtual Network Interface](https://github.com/zsvoboda/fadoc/blob/main/src/img/vlan/edit.lacp.vlan.vif.png)

Specify the `Netmask` and `Gateway` values, and enable the new file virtual interface.

![Edit Virtual Network Interface Dialog](https://github.com/zsvoboda/fadoc/blob/main/src/img/vif.physical/edit.file.vif.dialog.png)

If you no longer need the file virtual interface, you can delete it by clicking the `Delete` command in the Ethernet list.

![Delete File Virtual Network Interface](https://github.com/zsvoboda/fadoc/blob/main/src/img/vlan/delete.lacp.file.vif.png)

You can also delete the VLAN virtual network interfaces by clicking the `x` button next to a specific interface in the `Subnets` widget on the `Settings/NETWORK/Configuration` page.

![Delete VLAN Virtual Network Interface](https://github.com/zsvoboda/fadoc/blob/main/src/img/vlan/delete.vlan.lacp.png)

Finally, you can also delete the VLAN subnet by clicking on the `delete` icon in the `Subnets` widget on the `Settings/NETWORK/Configuration` page  if it is no longer needed.

![Delete Subnet](https://github.com/zsvoboda/fadoc/blob/main/src/img/vlan/delete.subnet.png)# Creating Virtual File Interface using VLAN and CLI
Virtual LAN (VLAN) tagging is a networking technology that allows for the logical division of a network into multiple virtual networks, called VLANs, to improve network security, performance, and flexibility.

In FlashArray, VLANs are represented by subnets. You can create a new subnet called `101` using the `puresubnet` command:

```bash
puresubnet create --vlan 101 --prefix 192.168.1.0/24 --gateway 192.168.1.1 --mtu 9000 vlan101
```

Once the new subnet is created, you can create virtual network interfaces using the `purenetwork` command:

```bash
purenetwork eth create vif --subnet vlan101 ct0.eth4.101
purenetwork eth create vif --subnet vlan101 ct1.eth4.101
purenetwork eth create vif --subnet vlan101 ct0.eth5.101
purenetwork eth create vif --subnet vlan101 ct1.eth5.101
```

After running these commands, you'll see the new network interfaces: `ct0.eth4.101`, `ct1.eth4.101`, `ct0.eth5.101`, and `ct1.eth5.101`, which can be used to create a new virtual file interface:

```bash
purenetwork eth create vif --address 192.168.1.231 --subinterfacelist ct0.eth4.101,ct1.eth4.101,ct0.eth5.101,ct1.eth5.101 filevif0
purenetwork eth setattr --netmask 255.255.255.0 --gateway 192.168.1.1 filevif0
purenetwork eth enable filevif0
``` 

If you no longer need the file virtual interface, you can delete it with the following commands:

```bash
purenetwork eth delete filevif0
purenetwork eth delete ct0.eth4.101,ct1.eth4.101,ct0.eth5.101,ct1.eth5.101

```

Finally, you can also delete the VLAN subnet if it is no longer needed.

```bash
puresubnet delete vlan101
```

# Creating a File Virtual Interface using LACP with VLAN in the CLI

Alternatively, you can create a new file virtual interface from LACP bonds with VLAN subnet:

```bash
purenetwork eth create lacpbond --subinterfacelist ct0.eth4,ct0.eth5 lacp0
purenetwork eth create lacpbond --subinterfacelist ct1.eth4,ct1.eth5 lacp1
purenetwork eth enable lacp0
purenetwork eth enable lacp1
```

Then create LACP bond interfaces with the VLAN 

```bash
purenetwork eth create vif --subnet vlan101 lacp0.101
purenetwork eth create vif --subnet vlan101 lacp1.101
```

Then, create a new virtual file interface using the newly created LACP bonds:

```bash
purenetwork eth create vif --address 192.168.1.231 --subinterfacelist lacp0.101,lacp1.101 --servicelist file filevif0
purenetwork eth setattr --netmask 255.255.255.0 --gateway 192.168.1.1 filevif0
purenetwork eth enable filevif0
``` 

If you no longer need the file virtual interface, you can delete it with the following commands:

```bash
purenetwork eth delete filevif0
purenetwork eth delete lacp0,lacp1
```

Finally, you can also delete the VLAN subnet if it is no longer needed.

```bash
puresubnet delete vlan101
```
# Configuring File Services DNS
FlashArray allows for mutiple DNS server configurations:

- **Management** DNS configuration is the default one and is used for resolution of all hostnames that array uses (e.g. SMTP servers, other arrays, etc.). If there is no File Services-specific DNS configuration, these DNS servers are used for resolution of File Services hostnames (e.g. hostnames in the Active Directory). FlashArray allows for only one management DNS configuration.
- **File** DNS configuration is used for resoultion of hostnames related to File Services. This File Services-specific DNS configuration often uses domain controller's DNS server that contains DNS records for all hostnames used in the Active Directory. FlashArray allows for only one file DNS configuration.
# Configuring File Services DNS using GUI

The DNS configuration widget is located on the `Settings/NETWORK/Configuration` page. By default, it contains the default management DNS configuration. If no additional configuration is specified, this DNS configuration is used for File Services.

![DNS list](https://github.com/zsvoboda/fadoc/blob/main/src/img/dns/dns.list.png)

You can edit the default DNS configuration; however, this is not recommended as it is likely used for array management.

![Management DNS](https://github.com/zsvoboda/fadoc/blob/main/src/img/dns/dns.mgmt.png)

In many cases, the default DNS server used by the array cannot resolve File Services-related hostnames defined in Active Directory. In such cases, you may need to use a different DNS configuration specifically for File Services.

To add a File Services-specific DNS configuration, click the `+` button in the DNS widget on the `Settings/NETWORK/Configuration` page.

![File Services DNS](https://github.com/zsvoboda/fadoc/blob/main/src/img/dns/dns.file.png)

Select the `file` checkbox for this DNS configuration.

**NOTE:** FlashArray allows only one `management` DNS configuration and one `file` DNS configuration.

The default network interface for communication with the DNS server is the `eth0` management Ethernet port. If this default network port cannot communicate with the File Services-specific DNS servers, you can use a different virtual network interface. In many cases, you would use your file virtual network interface, as the Windows domain server’s DNS and Active Directory are typically on the same network.# Configuring File Services DNS using CLI

The DNS configuration is managed via the `puredns` command.

```bash
puredns list
Name        Domain       Nameservers    Services    Source  CA certificate group  CA certificate
management  acmedns.com  10.7.32.53     management  -       -                     -
                         192.168.0.1                                              
                         192.168.0.101      
```

By default, this contains the management DNS configuration. If no additional configuration is specified, this DNS configuration is also used for File Services.

You can edit the default DNS configuration; however, it is not recommended, as it is likely used for array management.

In many cases, the default DNS server used by the array cannot resolve File Services-related hostnames defined in Active Directory. In such cases, a separate DNS configuration for File Services is necessary.

Use the `puredns` command to add a File Services-specific DNS configuration:

```bash
puredns create --domain acmedomain.com --nameservers 10.0.0.11,192.168.0.1,1
92.168.0.101 --services file --source filevif0 file
```

The default network interface for communication with the DNS server is the `eth0` Ethernet network port. If this default port cannot communicate with the File Services-specific DNS servers, you can specify a different virtual network interface. In many cases, the file virtual network interface is used, as the Windows domain server’s DNS and Active Directory are typically on the same network.

**NOTE**: FlashArray allows only one management DNS configuration and one file DNS configuration.

You can use the `delete` and `setattr` commands of `puredns` to delete or edit existing DNS configurations.# DNS Best Practices

By default, FlashArray sends domain name lookup requests via the management network interfaces, specifically through the `eth0` Ethernet network port, rather than the File Services virtual interface. While the File Services virtual interface can be used to route DNS lookups, it is not recommended.

As a best practice, Pure Storage recommends that client traffic to the FlashArray be routed over a separate IP network from management traffic. For example, the configuration below uses two different IP networks: one for NFS and SMB traffic (`192.168.2.x`) and the other (`10.x.x.x`) for management. The DNS server must be accessible from the `10.x.x.x` network but does not need to be accessible from the `192.168.2.x` network.

![DNS Best Practice](https://github.com/zsvoboda/fadoc/blob/main/src/img/dns/dns.best.practice.png)

Regardless of which interface is being used for DNS services, that interface must be able to reach the DNS server. You can test whether DNS services are correctly configured and available by running the following command:

```bash
puredns lookup <hostname>
```

If it returns an IP address, DNS services have been correctly configured.# Creating DNS A-Record

The final step in configuring FlashArray networking is to manually create a DNS A-record for File Services. Use the IP address of the file virtual interface so that clients can access the array using its hostname.# File Services Users and Groups

By default, FlashArray uses local users and groups. Alternatively, you can join your array to an Active Directory or LDAP domain to perform user and group resolution via a directory server. Please note that FlashArray currently does not support simultaneous lookups in both Active Directory and LDAP. Additionally, the SMB protocol does not support the LDAP directory server.

The local users and groups database serves as a fallback for directory service resolution. If a user or group cannot be found in the directory server, FlashArray will attempt to resolve it in the local user database.

For NFS protocol, there is a `User Mapping` configuration option. If this option is set to false, FlashArray will not perform any user lookup in either the local user database or the directory server (Active Directory or LDAP).# Array Local Users and Groups

FlashArray File Local Users and Groups is a feature that allows for a locally stored directory of users and groups, providing an alternative to external authentication solutions such as Active Directory or LDAP. By using local users and groups, clients can connect to the FlashArray File domain via SMB or NFS and authenticate with their respective credentials.

**NOTE:** For NFS, another option is to use an export policy with unmapped users (`--disable-user-mapping`). This allows local users on clients to access the NFS exports without user or group validation. FlashArray fully trusts the NFS client's user credentials (UID, GID, and secondary GIDs).

## Local Users

Each local user is a member of one primary group. Before creating a user, a primary group must exist for that user. A user can also be a member of other groups, referred to as secondary groups.

An admin should be a member of the Administrators group. Backup operators should be members of the Backup Operators group.

FlashArray contains the following built-in users, which cannot be modified or removed:

- **Administrator** (uid=0) is used for array configuration and management.
- **Guest** (uid=65534) is used for anonymous access. This account is disabled by default.

**NOTE:** NFS squashing capability changes the user identity to the anonymous user. The default uid for the anonymous user in NFS policy rules is 65534. Ensure that the user with uid 65534 is defined. You can create such a user in Active Directory or LDAP or rely on the `Guest` local user. In this case, you must explicitly enable the `Guest` user.

## Local Groups

A local group can have many members, but only local users can be members of a local group (not other groups). Before deleting a local group, all members must be removed from the group.

Local groups can also contain external users (from Active Directory or LDAP).

FlashArray contains the following built-in groups, which cannot be modified or removed:

- **Administrators** (gid=0) contains users with administrative privileges.
- **Guests** (gid=65534) contains guest users.
- **Backup Operators** contains users responsible for backup and archival tasks.

## Managing Local Users and Groups using GUI

Navigate to the `Settings/Access/File Services` configuration page to review the local users and groups.

![Local Users and Groups](https://github.com/zsvoboda/fadoc/blob/main/src/img/local.users/file.services.page.png)

## Managing Local Users and Groups using CLI

Please refer to the `pureds` command in the CLI reference documentation.

```bash
pureds local user -h
usage: pureds local user [-h] {add,create,delete,disable,enable,list,remove,rename,setattr} ...

positional arguments:
  {add,create,delete,disable,enable,list,remove,rename,setattr}
    list                list users
    create              create one or more users
    delete              delete one or more users
    add                 add the user to one or more groups
    remove              remove the user from one or more groups
    enable              enable one or more users
    disable             disable one or more users
    rename              rename user
    setattr             set attributes of the user 
```

Here are a few examples of how to use the pureds command:

List all local users:

```bash
pureds local user list
```

List a local group with a specific SID:

```bash
pureds local group list --sid S-1-5-32-544
```

Create a new local user:

1. Use the `pureds local user create` command to create a new user. 

```bash
pureds local user create <username>
```

Replace <username> with the desired username for the new user.

2. Set additional attributes for the user, if needed: 

```bash
pureds local user setattr <username> --primary-group <groupname> --password --uid <uid> --email <email>
```

Replace <username>, <groupname>, <uid>, and <email> with the appropriate values.

3. Add the user to one or more groups:

```bash
pureds local user add <username> --group <groupname>
```

Replace <username> and <groupname> with the appropriate values.

4. Enable the user (if they are not enabled by default):

```bash
pureds local user enable <username>
```

5. Verify the user creation by listing the users:

```bash
pureds local user list
```

## Referencing Local Users and Groups from Clients

SMB clients refer to local users using their local username and the domain name 'domain'. For example, when mapping a drive from Windows, use the `domain\<local-user-name>` format.

![Windows Map Drive](https://github.com/zsvoboda/fadoc/blob/main/src/img/local.users/windows.map.dialog.png)
![Windows User Login](https://github.com/zsvoboda/fadoc/blob/main/src/img/local.users/windows.login.dialog.png)

In this example, `zsvoboda` is the local user that has been previously created on the array. This user is a member of the `Administrators` group.

To execute Computer Management on Windows, use the following command:

```cmd
runas /noprofile /netonly /user:domain\zsvoboda "mmc compmgmt.msc"
```
**NOTE:** You must execute the Windows command prompt with the `Run as administrator` option. 

![Windows Computer Management](https://github.com/zsvoboda/fadoc/blob/main/src/img/local.users/computer.management.png)

Connect to the FlashArray using the file virtual interface IP address:

![Windows Computer Management Connect](https://github.com/zsvoboda/fadoc/blob/main/src/img/local.users/computer.management.login.png)

From there, you can explore the array settings, such as SMB shares, sessions, open files, or array local users:

![Windows Computer Management Users](https://github.com/zsvoboda/fadoc/blob/main/src/img/local.users/computer.management.users.png)

Similarly, when mounting an SMB share from a Linux client, use the following command: 

```bash
mount -t cifs -o user=zsvoboda,domain=domain,vers=3,uid=1000 //192.168.1.231/home /mnt/home
```

On the Windows command line, use:

```cmd
net use Z: \\192.168.1.231\home /user:domain\zsvoboda
```

### Changing Windows Security Settings (ACL)

When setting Windows share, directory, or individual file permissions (also known as ACLs), refer to local users using the format `<file-virtual-network-ip-or-hostname>\<local-user_name>`.

For example:

1. Right-click on the Windows mapped drive and select `Properties` from the drop-down menu.

![Windows Explorer Mapped Drive Properties](https://github.com/zsvoboda/fadoc/blob/main/src/img/local.users/explorer.share.png)

2. In the properties dialog, navigate to the `Security` tab and click on the `Edit` button to add a new user to the list.

![Security Properties](https://github.com/zsvoboda/fadoc/blob/main/src/img/local.users/security.properties.png)

3. Click the `Add` button to add a FlashArray local user to the list.

![Security Permissions](https://github.com/zsvoboda/fadoc/blob/main/src/img/local.users/permissions.png)

4. Refer to the `zsvoboda` local user as `192.168.1.231\zsvoboda`.

![Users](https://github.com/zsvoboda/fadoc/blob/main/src/img/local.users/users.png)

5. Change the share access permissions for the array local user `zsvoboda`.

![Full Access](https://github.com/zsvoboda/fadoc/blob/main/src/img/local.users/permissions.full.access.png)
# Joining Active Directory Domain

FlashArray can authenticate SMB and NFS users against an Active Directory domain. To do this, you must first create an account for the FlashArray in your target Active Directory domain.

The `Active Directory Accounts` configuration widget is located on the `Settings/ACCESS/Users And Policies` page. 

![DNS list](https://github.com/zsvoboda/fadoc/blob/main/src/img/ad/ad.list.png)

Click on the `+` button in the top right corner of the widget.

You can either create a new computer account for your array:

![AD New Computer](https://github.com/zsvoboda/fadoc/blob/main/src/img/ad/ad.new.account.png)

or use an existing computer account in your Active Directory:

![AD Existing Computer](https://github.com/zsvoboda/fadoc/blob/main/src/img/ad/ad.existing.account.png)

## Joining LDAP

Alternatively, you can also join an LDAP domain from the `Directory Service` widget on the same configuration page. 

Be aware that FlashArray currently supports the LDAP authentication only for the NFS protocol. Also LDAP and Active Directory can't be used at the same time.
