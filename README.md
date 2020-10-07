# SharedMATLABEngine

SharedMATLABEngine allows MATLAB and Julia to share data with an open MATLAB session through a MATLAB command line embedded in the Julia REPL. The functionality is similar to [MATLAB.jl](https://github.com/JuliaInterop/MATLAB.jl). The main advantage of SharedMATLABEngine.jl over MATLAB.jl is the ability to connect to an already open named MATLAB engine session. The advantages of MATLAB.jl over SharedMATLABEngine.jl are... well a lot. A lot more effort has gone into that project than this one. Unless you really need to connect to an already open MATLAB session, you should probably use MATLAB.jl.

SharedMATLABEngine goes through a PyCall and the uses the matlab.engine interface in Python.

## Installation
Since SharedMATLABEngine connects through the Python API for the MATLAB Engine, [Python must first be installed](https://www.python.org/downloads/).

Once Python is installed, the MATLAB Engine API for Python needs to be set up. In the MATLAB command prompt, enter:
```matlab
>> cd (fullfile(matlabroot,'extern','engines','python'))

>> system('python setup.py install')
```

Finally, SharedMATLABEngine must be installed in Julia. To do so, press the `]` key in the Julia REPL to enter Pkg mode. You should see the command prompt change from `julia>` to `(@v1.x) pkg>`. Since SharedMATLABEngine is not registered, it must be added by entering:
```julia
(@v1.5) pkg> add https://github.com/jonniedie/SharedMATLABEngine.jl
```

From here, SharedMATLABEngine should work. If not, check out the [documentation for installing the MATLAB Engine API in Python](https://www.mathworks.com/help/matlab/matlab_external/install-the-matlab-engine-for-python.html).


## Getting Started
To begin using SharedMATLABEngine, import the library and call the function `connect_matlab(engine_name)`. To see a list of available MATLAB sessions, call `find_matlab()`. Alternitavely, you can call `matlab.engine.engineName` in MATLAB to see the name of that specific session.

```julia
julia> using SharedMATLABEngine

julia> eng = connect_matlab("MATLAB_25596"); # Get from matlab.engine.engineName in MATLAB
REPL mode SharedMATLABEngine initialized. Press > to enter and backspace to exit.
```

Now the MATLAB command line can be accessed from the Julia REPL by typing `>`.
```matlab
>> a = magic(3)

a =

     8     1     6
     3     5     7
     4     9     2
```

Julia variables can be interpolated into MATLAB commands via the `$` operator.
```matlab
>> $a21 = a(2,1)
3.0

>> b = {zeros($a21), 'some words'}

b =

  1×2 cell array

    {3×3 double}    {'some words'}


>> [$z, c] = b{:}

c =

    'some words'

julia> (a21, z)
(3.0, [0.0 0.0 0.0; 0.0 0.0 0.0; 0.0 0.0 0.0])
```

MATLAB command outputs can be accessed in Julia through the `mat"` string macro or the `Engine` object's `workspace` field.
```julia
julia> b = a21 .+ mat"a"
3×3 Array{Float64,2}:
 11.0   4.0   9.0
  6.0   8.0  10.0
  7.0  12.0   5.0

julia> sqrt.(eng.workspace.a)
3×3 Array{Float64,2}:
 2.82843  1.0      2.44949
 1.73205  2.23607  2.64575
 2.0      3.0      1.41421
```
