cc = meson.get_compiler('c')

# Find flex, configure lex generator
flex_cdata = configuration_data()

flex_min_version='2.5.31'
flex = find_program('flex', 'win_flex')

get_flex_version = find_program('get_flex_version.py')
flexversion_res = run_command([get_flex_version, flex], check: true)
flexversion = flexversion_res.stdout().strip()
if flexversion.version_compare('<' + flex_min_version)
  error('flex version @0@ >= @1@: NO'.format(flexversion, flex_min_version))
else
  message('flex version @0@ >= @1@: YES'.format(flexversion, flex_min_version))
endif

flex_cdata.set('FLEX', flex.path())
if cc.get_id() == 'msvc'
  flex_cdata.set('FLEX_ARGS', '--nounistd')
else
  flex_cdata.set('FLEX_ARGS', '')
endif

gen_lex = configure_file(input : 'gen_lex.py.in',
  output : 'gen_lex.py',
  configuration : flex_cdata)

# Find bison, configure grammar generator
bison_cdata = configuration_data()

bison_min_version='2.4'
bison = find_program('bison', 'win_bison')

bversion_res = run_command([bison, '--version'])
if bversion_res.returncode() != 0
  error('Could not get bison version (@0@)'.format(bversion_res.stderr()))
endif

bversion = bversion_res.stdout().split('\n')[0].split(' ')[-1].strip()
if bversion.version_compare('<' + bison_min_version)
  error('bison version @0@ >= @1@: NO'.format(bversion, bison_min_version))
else
  message('bison version @0@ >= @1@: YES'.format(bversion, bison_min_version))
endif



bison_cdata.set('BISON', bison.path())
bison_cdata.set('BISON_ARGS', '')

gen_grammar = configure_file(input : 'gen_grammar.py.in',
  output : 'gen_grammar.py',
  configuration : bison_cdata)

# Custom targets
parser = custom_target('parselex',
  input : 'parse.l',
  output : ['lex.priv_gst_parse_yy.c', 'parse_lex.h'],
  command : [python3, gen_lex, '@OUTPUT0@', '@OUTPUT1@', '@INPUT@', 'DUMMY']
)

grammar = custom_target('parsegrammar',
  input : 'grammar.y',
  output : ['grammar.tab.c', 'grammar.tab.h'],
  command : [python3, gen_grammar, '@OUTPUT0@', '@OUTPUT1@', '@INPUT@'],
  depends : [parser],
)

gst_parse_sources += [parser, grammar]
