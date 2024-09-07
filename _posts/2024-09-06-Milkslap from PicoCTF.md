---
title: "MilkSlap from PicoCTF"
categories: [PicoCTF]
tags: [PicoCTF]
---
# MilkSlap from PicoCTF
[page](https://play.picoctf.org/practice/challenge/139?category=4&difficulty=2&page=2)
>Description
>[🥛](http://mercury.picoctf.net:7585/)

This is a site and there is `.PNG`photo so let's download it:
```
wget http://mercury.picoctf.net:7585/concat_v.png
```
It's a `.png` file so lets use `zsteg` on it:
```
$ zsteg a.png 
/var/lib/gems/3.1.0/gems/zpng-0.4.5/lib/zpng/scan_line.rb:303:in `upto': stack level too deep (SystemStackError)
	from /var/lib/gems/3.1.0/gems/zpng-0.4.5/lib/zpng/scan_line.rb:303:in `decoded_bytes'
	from /var/lib/gems/3.1.0/gems/zpng-0.4.5/lib/zpng/scan_line/mixins.rb:17:in `prev_scanline_byte'
	from /var/lib/gems/3.1.0/gems/zpng-0.4.5/lib/zpng/scan_line.rb:377:in `prev_scanline_byte'
	from /var/lib/gems/3.1.0/gems/zpng-0.4.5/lib/zpng/scan_line.rb:319:in `block in decoded_bytes'
	from /var/lib/gems/3.1.0/gems/zpng-0.4.5/lib/zpng/scan_line.rb:318:in `upto'
	from /var/lib/gems/3.1.0/gems/zpng-0.4.5/lib/zpng/scan_line.rb:318:in `decoded_bytes'
	from /var/lib/gems/3.1.0/gems/zpng-0.4.5/lib/zpng/scan_line/mixins.rb:17:in `prev_scanline_byte'
	from /var/lib/gems/3.1.0/gems/zpng-0.4.5/lib/zpng/scan_line.rb:377:in `prev_scanline_byte'
	 ... 9483 levels...
	from /var/lib/gems/3.1.0/gems/zsteg-0.2.13/lib/zsteg.rb:26:in `run'
	from /var/lib/gems/3.1.0/gems/zsteg-0.2.13/bin/zsteg:8:in `<top (required)>'
	from /usr/local/bin/zsteg:25:in `load'
	from /usr/local/bin/zsteg:25:in `<main>'
	
```
It doesn't work, I don't know why but i started searching on the internet and found the explanation:
```
$ RUBY_THREAD_VM_STACK_SIZE=500000000 zsteg a.png 
imagedata           .. text: "\n\n\n\n\n\n\t\t"
b1,b,lsb,xy         .. text: "picoCTF{imag3_m4n1pul4t10n_sl4p5}\n"
b1,bgr,lsb,xy       .. /var/lib/gems/3.1.0/gems/iostruct-0.1.2/lib/iostruct.rb:136:in `block in inspect': undefined method `type' for nil:NilClass (NoMethodError)

          when f.type == Integer
                ^^^^^
	from /var/lib/gems/3.1.0/gems/iostruct-0.1.2/lib/iostruct.rb:133:in `each'
	from /var/lib/gems/3.1.0/gems/iostruct-0.1.2/lib/iostruct.rb:133:in `map'
	from /var/lib/gems/3.1.0/gems/iostruct-0.1.2/lib/iostruct.rb:133:in `inspect'
	from /var/lib/gems/3.1.0/gems/zsteg-0.2.13/lib/zsteg/checker/wbstego.rb:41:in `to_s'
	from /var/lib/gems/3.1.0/gems/zsteg-0.2.13/lib/zsteg/checker.rb:291:in `puts'
	from /var/lib/gems/3.1.0/gems/zsteg-0.2.13/lib/zsteg/checker.rb:291:in `puts'
	from /var/lib/gems/3.1.0/gems/zsteg-0.2.13/lib/zsteg/checker.rb:291:in `show_result'
	from /var/lib/gems/3.1.0/gems/zsteg-0.2.13/lib/zsteg/checker.rb:326:in `process_result'
	from /var/lib/gems/3.1.0/gems/zsteg-0.2.13/lib/zsteg/checker.rb:271:in `check_channels'
	from /var/lib/gems/3.1.0/gems/zsteg-0.2.13/lib/zsteg/checker.rb:191:in `check_channels'
	from /var/lib/gems/3.1.0/gems/zsteg-0.2.13/lib/zsteg/checker.rb:118:in `block (4 levels) in check'
	from /var/lib/gems/3.1.0/gems/zsteg-0.2.13/lib/zsteg/checker.rb:118:in `each'
	from /var/lib/gems/3.1.0/gems/zsteg-0.2.13/lib/zsteg/checker.rb:118:in `block (3 levels) in check'
	from /var/lib/gems/3.1.0/gems/zsteg-0.2.13/lib/zsteg/checker.rb:99:in `each'
	from /var/lib/gems/3.1.0/gems/zsteg-0.2.13/lib/zsteg/checker.rb:99:in `block (2 levels) in check'
	from /var/lib/gems/3.1.0/gems/zsteg-0.2.13/lib/zsteg/checker.rb:98:in `each'
	from /var/lib/gems/3.1.0/gems/zsteg-0.2.13/lib/zsteg/checker.rb:98:in `block in check'
	from /var/lib/gems/3.1.0/gems/zsteg-0.2.13/lib/zsteg/checker.rb:97:in `each'
	from /var/lib/gems/3.1.0/gems/zsteg-0.2.13/lib/zsteg/checker.rb:97:in `check'
	from /var/lib/gems/3.1.0/gems/zsteg-0.2.13/lib/zsteg/cli/cli.rb:258:in `check'
	from /var/lib/gems/3.1.0/gems/zsteg-0.2.13/lib/zsteg/cli/cli.rb:172:in `block (2 levels) in run'
	from /var/lib/gems/3.1.0/gems/zsteg-0.2.13/lib/zsteg/cli/cli.rb:168:in `each'
	from /var/lib/gems/3.1.0/gems/zsteg-0.2.13/lib/zsteg/cli/cli.rb:168:in `block in run'
	from /var/lib/gems/3.1.0/gems/zsteg-0.2.13/lib/zsteg/cli/cli.rb:161:in `each'
	from /var/lib/gems/3.1.0/gems/zsteg-0.2.13/lib/zsteg/cli/cli.rb:161:in `each_with_index'
	from /var/lib/gems/3.1.0/gems/zsteg-0.2.13/lib/zsteg/cli/cli.rb:161:in `run'
	from /var/lib/gems/3.1.0/gems/zsteg-0.2.13/lib/zsteg.rb:26:in `run'
	from /var/lib/gems/3.1.0/gems/zsteg-0.2.13/bin/zsteg:8:in `<top (required)>'
	from /usr/local/bin/zsteg:25:in `load'
	from /usr/local/bin/zsteg:25:in `<main>'
```
***
And The flag is :
**picoCTF{imag3_m4n1pul4t10n_sl4p5}**
