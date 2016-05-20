#*******************************************************************************
#
#     SCons main construct file
#
#     Purpose: build KiCAD libraries
#
#     Toolkit:   Python3, YAML
#
#     Copyright (c) 2016, Harry E. Zhurov
#
#-------------------------------------------------------------------------------

import os
import sys
import glob
import subprocess
import re

#print( sys.version ) -> SCons does not support Python3 for now :(
#sys.path.append( os.path.join(os.environ['CAD'], 'kicad/tools') )
#import icgen

#===============================================================================
#
#     User definable area
#
#-------------------------------------------------------------------------------
#
#     Project settings
#
ToolDir      = os.environ['CAD'] + '/kicad/tools'
SchSrcDir    = 'sch/src'
                
#-------------------------------------------------------------------------------
#
#    Project structure
#
SchLibDir   = 'sch/ic'
SchCmpDir   = os.path.join(SchLibDir, 'cmp')

ServiceDirs = [ SchLibDir, SchCmpDir ]
for i in ServiceDirs:
    curdir = os.path.join( os.getcwd(), i )
    if not os.path.exists(curdir):
        print 'Directory ' + curdir + ' does not exists. Creating the directory...',
        os.mkdir(curdir)
        print 'done'

#-------------------------------------------------------------------------------
#
#    Project specific stuff
#
SrcExt     = 'yml'
LibExt     = 'lib'
LibDocExt  = 'dcm'
CmpExt     = 'cmp'
CmpDocExt  = 'dcmp'

#-------------------------------------------------------------------------------
#
#      Tools
#

CONNGEN  = os.path.join(ToolDir, 'conngen.py')
ICGEN    = os.path.join(ToolDir, 'icgen.py')
                 
#-------------------------------------------------------------------------------
#
#    Service stuff
#
SEP_LEN = 76

#-------------------------------------------------------------------------------
def namegen(fullpath, ext):
    basename = os.path.basename(fullpath)
    name     = os.path.splitext(basename)[0]
    return name + os.path.extsep + ext
#-------------------------------------------------------------------------------
def pexec(cmd):
    p = subprocess.Popen( cmd.split(), universal_newlines = True,
                         stdin  = subprocess.PIPE,
                         stdout = subprocess.PIPE,
                         stderr = subprocess.PIPE )


    out, err = p.communicate()

    return p.returncode, out, err
#-------------------------------------------------------------------------------
#
#    Build .cmp and .dcmp files from yml source file
#
def create_cmp(target, source, env):
    #------------------------------------------------
    #
    #   Check if the first source is compiling
    #
    if env['FIRST_ENTRY'] == False:
        env['FIRST_ENTRY'] = True
        print '*'*SEP_LEN

    #------------------------------------------------
    #
    #   Launch component generator
    #
    src_name = str(source[0])
    dir_name = os.path.dirname(src_name)
    lib_name = os.path.split(dir_name)[1]
    cmp_dir  = os.path.join(SchCmpDir, lib_name)

    if not os.path.exists(cmp_dir):
        os.mkdir(cmp_dir)

    print 'create component: \'' + os.path.basename( str(target[0]) ) + '\''
    cmd = env['ICGEN'] + ' -o ' + cmp_dir + ' -s ' + str(source[0])
    rcode, out, err = pexec(cmd)
    out += err
    if out:        print out.replace('`', '\'')
    if rcode != 0: return rcode

#-------------------------------------------------------------------------------
#
#    Build executable file from object files
#
def build_lib(target, source, env):
    if env['FIRST_ENTRY'] == False:
        print '*'*SEP_LEN
        print 'sources are up to date'

    print '-'*16 #SEP_LEN
    print 'create library:   \'' + os.path.basename( str(target[0]) ) + '\''

    src_list = []
    for i in xrange(len(source)):
        src_list.append( str(source[i]) )

    doc_list = []
    for i in xrange(len(source)):
        doc_file = namegen( str(source[i]), CmpDocExt)
        doc_list.append( os.path.join( os.path.dirname(str(source[i])), doc_file) ) 
        
    #-------------------------------------------------------------
    #
    #    Create library file
    #
    lib   = 'EESchema-LIBRARY Version 2.3' + os.linesep
    lib  += '#encoding utf-8' + os.linesep
    tail  = '#' + os.linesep
    tail += '#End Library' + os.linesep
    
    for i in src_list:
        f = open(i, 'rb')
        lib += f.read()
        f.close()
        
    lib += tail
    
    with open( str(target[0]), 'wb') as f:
        f.write(lib)

    #-------------------------------------------------------------
    #
    #    Create doc-lib file
    #
    libdoc   = 'EESchema-DOCLIB  Version 2.0' + os.linesep
    tail  = '#' + os.linesep
    tail += '#End Doc Library' + os.linesep
    
    for i in doc_list:
        f = open(i, 'rb')
        libdoc += f.read()
        f.close()
        
    libdoc += tail
    
    libdoc_name = namegen( str(target[0]), LibDocExt)
    
    with open( os.path.join( os.path.dirname(str(target[0])), libdoc_name), 'wb') as f:
        f.write(libdoc)

    print '*'*SEP_LEN

#-------------------------------------------------------------------------------

#-------------------------------------------------------------------------------
#
#   Builder objects
#
#-------------------------------------------------------------------------------
yml2cmp    = Builder(action         = create_cmp,
                     suffix         = CmpExt,
                     src_suffix     = SrcExt)
#-------------------------------------------------------------------------------
cmp2lib    = Builder(action         = build_lib,
                     suffix         = LibExt,
                     src_suffix     = CmpExt)
#-------------------------------------------------------------------------------

#-------------------------------------------------------------------------------
#
#    State variables
#
FirstEntry = False
LibDone    = True

#-------------------------------------------------------------------------------
#
#    Environment
#

env = Environment(TOOLS = {})

env['BUILDERS'] = {
                     'yml2cmp' : yml2cmp,
                     'cmp2lib' : cmp2lib
                  }

env['CONNGEN'    ] = CONNGEN
env['ICGEN'      ] = ICGEN
env['FIRST_ENTRY'] = FirstEntry

#-------------------------------------------------------------------------------
#
#    Service functions
#
#-------------------------------------------------------------------------------
#
#   Make dictionary in form { src : cmp } from input source list
#
def make_target_dict(src_list, lib_name):
    targets = {}
    trg_dir = os.path.join(SchCmpDir, lib_name)
                            
    for i in src_list:
        name_ext = os.path.split(i)[1]
        name     = os.path.splitext(name_ext)[0] + '.' + CmpExt
        targets[i] = os.path.join(trg_dir, name)
        Depends(targets[i],  'SConstruct')      

    return targets

#-------------------------------------------------------------------------------
#
#   Make 'components from sources' nodes
#
def make_cmp(trg):
    olist = []

    for i in trg.items():
        olist.append( env.yml2cmp(i[1], i[0]) )

    return olist
    
#-------------------------------------------------------------------------------
#
#   Create components and libraries
#
src_dirs = glob.glob( os.path.join(SchSrcDir, '*') )
lib_trg_list = []         
for i in src_dirs:
    lib_name  = os.path.split(i)[1]
    src_files = glob.glob( os.path.join(i, '*.' + SrcExt) )           # get source list
    trg_dict  = make_target_dict(src_files, lib_name)                 # create source:target pairs
    cmp_list  = make_cmp(trg_dict)                                    # create component nodes
    lib_name  = os.path.join(SchLibDir, os.path.split(i)[1])          # generate library name
    lib_trg   = env.cmp2lib( source = cmp_list, target = lib_name )   # create library node
    lib_trg_list.append(lib_trg)                                      # add library node to target list

#-------------------------------------------------------------------------------
#
#    Clean and Rebuid
#
def remove_files(dir_, mask):
    file_list = glob.glob(dir_ + '/*.' + mask)
    for i in file_list:
        os.remove(i)

#-------------------------------------------------------------------------------
def clean(target, source, env):
    print 'Cleaning targets...',
    if os.path.exists('.sconsign.dblite'): os.remove('.sconsign.dblite')

    remove_files(SchCmpDir, CmpExt)
    remove_files(SchCmpDir, CmpDocExt)
    remove_files(SchLibDir, LibExt)
    print 'Done'

#-------------------------------------------------------------------------------
#
#       Targets
#
#-------------------------------------------------------
#
#   Intermediate targets
#
clean_all = env.Alias('cln', action = clean)
all       = lib_trg_list

#-------------------------------------------------------------------------------
#
#    Final targets
#
env.Alias('all', all)
env.Alias('rebuild', [clean_all, all])

env.AlwaysBuild('all', 'cln')

Default(all)

#-------------------------------------------------------------------------------
              
#===============================================================================

