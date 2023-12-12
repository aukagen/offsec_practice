Unsanitised vulerability


Piece of code which gave it away = ReconModel.php
```
<?php
#[AllowDynamicProperties]

class ReconModel
{   
    public function __construct($ip)
    {
        $this->ip = $ip;
    }

    public function getOutput()
    {
        # Do I need to sanitize user input before passing it to shell_exec?
        return shell_exec('ping -c 3 '.$this->ip);
    }
}         
```

All that needed ot be done wa append a `& ls` to the code, and a directory listing was diplayed after the ping. Not difficult to find the flag after that.