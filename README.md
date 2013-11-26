nominate
========

The 'nominate' gem, to run W- and DW-NOMINATE from Ruby


Requirements
------------

* Linux
* R  
In Ubuntu: sudo apt-get install r-base r-base-dev
* GFortran (only for DW-NOMINATE)  
In Ubuntu: sudo apt-get install gfortran


Example
-------

Let's say you have a file of voting data in the format

> name1|party1|rollcall1 info|vote1  
> name2|party2|rollcall1 info|vote2  
> ...  
> name1|party1|rollcall2 info|vote1  
> name2|party2|rollcall2 info|vote2  
> ...

which is similar to what you can download from the
<a href="http://www.gencourt.state.nh.us/downloads/">New Hampshire state legislature</a>.
And these are collected in file '1999NHrollcalls.csv' and so forth.

The following code will read the rollcall data into WSession objects, and add them to a DWChamber object.
chamber.dwnominate starts the nominate process, first calling wnominate on each WSession, then using the
results to write the input files for the DW-NOMINATE program, and then running DW-NOMINATE. All results,
including output graphs from the R wnominate package, will be stored in a folder called 'nominate'.

An alternative is to call wnominate(prefix) on WSession objects, which will run only the W-NOMINATE program,
giving each output file the specified prefix.



<pre>
require 'nominate'

chamber = DWChamber.new

# votes other than Yes or No are counted as missing ('M')
vote_key = Hash.new('M')
vote_key['Yes'] = 'Y'
vote_key['No'] = 'N'

['1999', '2001', '2003', '2005', '2007'].each do |year|
  year = (year.to_i + 1).to_s
  file = year + 'NHrollcalls.csv'
  lines = IO.readlines(file)
  session = WSession.new
  # default party is 'unknown'
  parties = Hash.new('unknown')
  old_issue = 0
  # default vote is 'M' for 'missing'
  rollcall = Hash.new('M')
  lines.each_with_index do |line, i|
    data = line.split('|')
    name = data[0]
    issue = data[2]
    vote = vote_key[data[3].chomp]
    if issue != old_issue
      session.add_rollcall(rollcall) unless i == 0
      rollcall = Hash.new('M')
      old_issue = issue
    end
    rollcall[name] = vote
    parties[name] = data[1] if not parties.has_key?(name)
    # get the last rollcall when you reach the end of the file
    session.add_rollcall(rollcall) if i == lines.length - 1
  end
  session.parties = parties
  chamber.add_session(session)
end

chamber.dwnominate
</pre>

