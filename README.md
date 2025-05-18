# Battery-Information-on-Proxmox

## 🔋 Overview

This project allows you to display real-time battery information in the **Proxmox VE Web UI** — useful for laptops or battery-backed systems.

Tested on: **Proxmox VE 8.4.0**

---

## 📸 Screenshot

![Battery Info in Proxmox UI](./img/battery%20information.png)

---

## 🚀 Quick Installation (Proxmox 8.4.0 Only)

Replace the original files and install `acpi`:

```bash
# Replace backend API
curl -o /usr/share/perl5/PVE/API2/Nodes.pm https://raw.githubusercontent.com/dbghelp/Battery-Information-on-Proxmox/main/Nodes.pm

# Replace frontend JavaScript
curl -o /usr/share/pve-manager/js/pvemanagerlib.js https://raw.githubusercontent.com/dbghelp/Battery-Information-on-Proxmox/main/pvemanagerlib.js

# Install battery reporting tool
apt install acpi

# Restart web service
systemctl restart pveproxy
```

---

## For other versions
Modify ```/usr/share/perl5/PVE/API2/Nodes.pm```, search and add in the following code in highlighted in green:
```diff
	$res->{swap} = {
	    free => $meminfo->{swapfree},
	    total => $meminfo->{swaptotal},
	    used => $meminfo->{swapused},
	};

	$res->{pveversion} = PVE::pvecfg::package() . "/" .
	    PVE::pvecfg::version_text();
		
+       $res->{battery_info} = `acpi -b 2>&1`; 

	my $dinfo = df('/', 1);     # output is bytes

	$res->{rootfs} = {
	    total => $dinfo->{blocks},
	    avail => $dinfo->{bavail},
	    used => $dinfo->{used},
	    free => $dinfo->{blocks} - $dinfo->{used},
	};

	return $res;
    }});
```

Modify ```/usr/share/pve-manager/js/pvemanagerlib.js```, search and add in the following code highlighted in green:

```diff
	{
	    itemId: 'version',
	    colspan: 2,
	    printBar: false,
	    title: gettext('Manager Version'),
	    textField: 'pveversion',
	    value: '',
	},
+	{
+	    itemId: 'battery_info',
+	    colspan: 2,
+	    printBar: false,
+	    title: gettext('Battery information'),
+	    textField: 'battery_info',
+	    value: '',
+	},	
    ],

    updateTitle: function() {
	var me = this;
	var uptime = Proxmox.Utils.render_uptime(me.getRecordValue('uptime'));
	me.setTitle(me.pveSelNode.data.node + ' (' + gettext('Uptime') + ': ' + uptime + ')');
    },
```
