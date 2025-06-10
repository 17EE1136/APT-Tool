Project Description
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
PenTest Automation CLI is a lightweight, modular framework for automating the four main phases of a penetration test—Reconnaissance, Exploitation, Cracking, and Post-Exploitation—entirely from the command line. Each phase lives in its own Python module, and the central main.py script ties them together behind a simple, interactive menu or fully-scriptable CLI.

How It Works

  1. Module Structure
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
       > reconnaissance.py
            Runs tools like Amass, Nmap, Nslookup, Whois, Recon-ng, Sublist3r, and Nikto in 
            parallel against one or more targets, saving raw output under results/<target>/reconnaissance/.
        
       > exploitation.py
            Automates SQLmap, Metasploit, Shellshock curl tests, Gobuster, FFUF, and Dirb scans—optionally
            prompting once for a custom wordlist—and writes results to results/<target>/exploitation/.
        
       > cracking.py
            Supports John the Ripper, Hashcat, Medusa, and Hydra. The script auto-detects whether each target
            is a hash file or a host, then runs the appropriate tools in parallel with a user-specified (or default) wordlist, logging   
            everything to results/<target>/cracking/.
        
       > post_exploitation.py
            Launches reverse shells, Meterpreter handlers, LinPEAS, WinPEAS, SMBclient, and SMBmap—each wrapped 
            via the Unix script utility so you capture live, colored output to results/<target>/post_exploitation/.
    

  2. Concurrency & Batch Mode
 ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

        > All modules use Python’s concurrent.futures.ThreadPoolExecutor to run each tool, or each target, in its own thread.
        
        > The core function batch_run(targets, func, tools) in main.py will spin up one thread per target, 
          calling your chosen func(target, tools)—so you can recon multiple IPs at once, or crack multiple hash files simultaneously.
        
        > You can even mix-and-match: by calling batch_run([...], run_exploitation_tools, [...]) you can 
          run exploit and recon modules in the same pool of threads if you like.

  3. Interactive & Scriptable
 ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

        > Interactive mode via menu prompts lets you pick exactly which tools and targets you want.
        
        > Non-interactive mode (for CI/CD or automation) by invoking the entrypoint with flags 
        (e.g. python main.py recon example.com —tools Amass Nmap) once you’ve extended it with argparse.

  4. Results Organization
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
      > Every run dumps raw logs and outputs into a neat folder hierarchy under results/<target>/<phase>/<tool>.txt.
      
      > This makes it trivial to post-process, grep, or integrate into a larger reporting pipeline.


Extending the Framework with Your Own Tool

You can add any custom tool—whether a home-grown script or a third-party scanner—in just three steps:

  1. Create a “run_…” Function
      In the module for the relevant phase (e.g. modules/reconnaissance.py), add:
       python
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
        def run_mytool(target, <maybe wordlist?>):
            """
            Brief description of *why* we run this tool:
            - what kind of data it gathers
            - why it’s useful in Recon or Exploit, etc.
            """
            outfile = f"results/{target}/reconnaissance/mytool.txt"
            cmd     = f"mytool --target {target} --output {outfile}"
            subprocess.run(cmd, shell=True)

            
  2. Register It in the Mapping
      Still in the same file, find the tools_mapping dict and add:
      python
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
        tools_mapping["MyTool"] = run_mytool

        
  3. Expose It in the Menu
      In main.py, update the tool_options for that phase:
      python
 ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------    
    "Reconnaissance": [
      "Amass", "Nmap", /* … */, "Nikto",
      "MyTool"   # ← your new entry
    ],

    
After that, MyTool will appear in the interactive menu under Reconnaissance. Running:
bash
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
    Enter target domains/IPs: example.com
    Select tools for Reconnaissance:
    1. Amass
    …
    8. MyTool
    Enter tool numbers: 8
will execute your new function, saving its output alongside the others.

Contributing on GitHub

    1. Fork this repository to your own GitHub account.
    
    2. Create a new branch (git checkout -b add-mytool).
    
    3. Implement your changes following the three steps above.
    
    4. Test locally by running your new tool against a known target.
    
    5. Commit & push your branch.
    
    6. Open a Pull Request back to main, describing your new tool and its benefits.
    
    We’ll review, merge, and ship your contribution so other users can benefit from your addition!

With this design you get a truly modular, parallel, and extensible Pentest Automation toolkit—easy to script, easy to expand, and perfect for both red-team workflows and CI/CD security gates.






