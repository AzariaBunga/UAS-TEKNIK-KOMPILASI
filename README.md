# UAS-TEKNIK-KOMPILASI
uas
Tahapan yang disimulasikan:
1. Analisis Leksikal -> memecah source code menjadi token
2. Analisis Sintaksis -> membangun Abstract Syntax Tree (AST)
3. Analisis Semantik -> pengecekan deklarasi variabel & tipe data
4. Generasi Kode Antara -> menghasilkan Three-Address Code (TAC)
Bahasa mini yang didukung (lihat dokumentasi untuk grammar BNF lengkap):
int i;
int sum;
i = 0;
sum = 0;
while (i < 5) {
sum = sum + i;
i = i + 1;
}
=====================================================================
"""
import re
# =====================================================================
# TAHAP 1: ANALISIS LEKSIKAL
# =====================================================================
class Lexeme:
"""Representasi satu token hasil scanning.
def
init
(self, kind, text, row):
__
__
self.kind = kind
self.text = text
self.row = row
"""
def
__
repr
(self):
__
return f"[{self.kind} '{self.text}']"
# Aturan token diurutkan dari yang paling spesifik ke paling umum
RULES = (
("NUMBER"
, r"\d+(\.\d+)?"),
("WHILE"
, r"\bwhile\b"),
("INT"
, r"\bint\b"),
("FLOAT"
, r"\bfloat\b"),
("NAME"
, r"[A-Za-z
][A-Za-z0-9
_
_
("RELOP"
, r"
==|!=|<=|>=|<|>"),
("ASSIGN"
, r"
=
"),
("ADD"
, r"\+"),
("SUB"
, r"
-
"),
("MUL"
, r"\*"),
("DIV"
, r"/"),
("LPAR"
, r"\("),
("RPAR"
, r"\)"),
("LBRACE"
, r"\{"),
("RBRACE"
, r"\}"),
("SEMI"
, r";"),
("NEWLINE"
, r"\n"),
("BLANK"
, r"[ \t]+"),
("UNKNOWN"
, r"
.
"),
]*"),
)
SCANNER = "|"
RESERVED = {"while"
.join(f"(?P<{name}>{pat})" for name, pat in RULES)
,
"int"
,
"float"}
def scan(source):
"""Analisis leksikal: mengubah teks program menjadi daftar Lexeme.
result = []
row = 1
for m in re.finditer(SCANNER, source):
kind = m.lastgroup
text = m.group()
"""
if kind == "NEWLINE":
row += 1
continue
if kind == "BLANK":
continue
if kind == "UNKNOWN":
raise SyntaxError(f"[LEKSIKAL] Simbol tidak dikenal {text!r} di baris {row}")
if kind == "NAME" and text in RESERVED:
kind = text.upper()
result.append(Lexeme(kind, text, row))
result.append(Lexeme("EOF"
return result
""
,
, row))
# =====================================================================
# TAHAP 2: ANALISIS SINTAKSIS -> ABSTRACT SYNTAX TREE
# =====================================================================
# Grammar BNF (recursive-descent, LL(1)):
#
# <program> ::= <stmt
list>
# # #
# # # list> "}"
_
#
# # # # =====================================================================
<stmt
list> _
_
::= { <stmt> }
<stmt> ::= <decl> | <assign> | <while
_
stmt>
<decl> ::= ("int" | "float") NAME ";"
<assign> ::= NAME "
=
" <expr> ";"
<while
stmt> _
::=
"while" "(" <cond> ")" "{" <stmt
<cond> ::= <expr> relop <expr>
<expr> ::= <term> { ("+" | "
-
") <term> }
<term> ::= <factor> { ("*" | "/") <factor> }
# <factor> ::= NAME | NUMBER | "(" <expr> ")"
class ASTNode:
pass
class ProgramNode(ASTNode):
def
init
__
__
(self, body):
self.body = body
class DeclNode(ASTNode):
def
init
__
__
(self, dtype, name, row):
self.dtype = dtype
self.name = name
self.row = row
class AssignNode(ASTNode):
def
init
(self, name, value, row):
__
__
self.name = name
self.value = value
self.row = row
class WhileNode(ASTNode):
def
init
__
__
(self, condition, body, row):
self.condition = condition
self.body = body
self.row = row
class ConditionNode(ASTNode):
def
init
__
__
(self, left, op, right, row):
self.left = left
self.op = op
self.right = right
self.row = row
class BinaryNode(ASTNode):
def
init
__
__
(self, left, op, right, row):
self.left = left
self.op = op
self.right = right
self.row = row
class NameNode(ASTNode):
def
init
(self, name, row):
__
__
self.name = name
self.row = row
class NumberNode(ASTNode):
def
init
(self, value, row):
__
__
self.value = value
self.row = row
class Parser:
"""Parser rekursif-turun yang membangun AST dari daftar Lexeme.
"""
def
init
(self, lexemes):
__
__
self.lexemes = lexemes
self.idx = 0
def peek(self):
return self.lexemes[self.idx]
def expect(self, kind):
tok = self.peek()
if tok.kind != kind:
raise SyntaxError(
f"[SINTAKSIS] Baris {tok.row}: mengharapkan '{kind}' "
f"tetapi menemukan '{tok.kind}' ({tok.text!r})"
)
self.idx += 1
return tok
def parse
_program(self):
body = self.parse
stmt
_
_
self.expect("EOF")
return ProgramNode(body)
list(stop_
at=("EOF"
,))
def parse
stmt
_
_
list(self, stop_
at):
stmts = []
while self.peek().kind not in stop_
at:
stmts.append(self.parse
stmt())
_
return stmts
def parse
stmt(self):
_
tok = self.peek()
if tok.kind in ("INT"
,
return self.parse
"FLOAT"):
decl()
_
if tok.kind == "NAME":
return self.parse
assign()
_
if tok.kind == "WHILE":
return self.parse
while()
_
raise SyntaxError(f"[SINTAKSIS] Baris {tok.row}: statement tidak dikenal ({tok.kind})")
def parse
decl(self):
_
type
tok = self.peek()
_
self.expect(type
tok.kind)
_
name
tok = self.expect("NAME")
_
self.expect("SEMI")
return DeclNode(type
tok.text, name
tok.text, type
tok.row)
_
_
_
def parse
assign(self):
_
name
tok = self.expect("NAME")
_
self.expect("ASSIGN")
value = self.parse
expr()
_
self.expect("SEMI")
return AssignNode(name
tok.text, value, name
tok.row)
_
_
def parse
while(self):
_
w
tok = self.expect("WHILE")
_
self.expect("LPAR")
cond = self.parse
cond()
_
self.expect("RPAR")
self.expect("LBRACE")
body = self.parse
stmt
_
_
list(stop_
at=("RBRACE"
self.expect("RBRACE")
return WhileNode(cond, body, w
tok.row)
_
,))
def parse
cond(self):
_
left = self.parse
expr()
_
op_
tok = self.expect("RELOP")
right = self.parse
expr()
_
return ConditionNode(left, op_
tok.text, right, op_
tok.row)
def parse
expr(self):
_
node = self.parse
term()
_
while self.peek().kind in ("ADD"
,
"SUB"):
op_
tok = self.peek()
self.expect(op_
tok.kind)
right = self.parse
term()
_
node = BinaryNode(node, op_
tok.text, right, op_
tok.row)
return node
def parse
term(self):
_
node = self.parse
factor()
_
while self.peek().kind in ("MUL"
,
"DIV"):
op_
tok = self.peek()
self.expect(op_
tok.kind)
right = self.parse
factor()
_
node = BinaryNode(node, op_
tok.text, right, op_
tok.row)
return node
def parse
factor(self):
_
tok = self.peek()
if tok.kind == "NUMBER":
self.expect("NUMBER")
value = float(tok.text) if "
.
return NumberNode(value, tok.row)
if tok.kind == "NAME":
self.expect("NAME")
return NameNode(tok.text, tok.row)
if tok.kind == "LPAR":
self.expect("LPAR")
node = self.parse
expr()
_
self.expect("RPAR")
return node
" in tok.text else int(tok.text)
raise SyntaxError(f"[SINTAKSIS] Baris {tok.row}: ekspresi tidak valid ({tok.kind})")
def render
ast(node, depth=0):
_
"""Mencetak AST sebagai pohon berindentasi.
"""
pad = " " * depth
if isinstance(node, ProgramNode):
print(f"{pad}ProgramNode")
for s in node.body:
render
ast(s, depth + 1)
_
elif isinstance(node, DeclNode):
print(f"{pad}DeclNode({node.dtype} {node.name})")
elif isinstance(node, AssignNode):
print(f"{pad}AssignNode({node.name} <-)")
render
ast(node.value, depth + 1)
_
elif isinstance(node, WhileNode):
print(f"{pad}WhileNode")
print(f"{pad} Condition:")
render
ast(node.condition, depth + 2)
_
print(f"{pad} Body:")
for s in node.body:
render
ast(s, depth + 2)
_
elif isinstance(node, ConditionNode):
print(f"{pad}ConditionNode({node.op})")
render
ast(node.left, depth + 1)
_
render
ast(node.right, depth + 1)
_
elif isinstance(node, BinaryNode):
print(f"{pad}BinaryNode({node.op})")
render
ast(node.left, depth + 1)
_
render
ast(node.right, depth + 1)
_
elif isinstance(node, NameNode):
print(f"{pad}NameNode({node.name})")
elif isinstance(node, NumberNode):
print(f"{pad}NumberNode({node.value})")
# =====================================================================
# TAHAP 3: ANALISIS SEMANTIK
# =====================================================================
# Pengecekan yang dilakukan:
# a. Variabel tidak boleh dideklarasikan dua kali
# b. Variabel wajib dideklarasikan sebelum dipakai (assign / kondisi)
# c. Tipe hasil ekspresi harus kompatibel dengan tipe variabel tujuan
# (int <- int OK, float <- int/float OK, int <- float DITOLAK)
# d. Kondisi while harus tersusun dari operand yang valid & bertipe
# =====================================================================
class SemanticIssue(Exception):
pass
class SemanticChecker:
def
init
(self):
__
__
self.table = {} self.problems = []
# nama -> tipe
def check(self, program):
for stmt in program.body:
self.
check
stmt(stmt)
_
_
if self.problems:
raise SemanticIssue("\n"
return self.table
def
def
def
.join(self.problems))
check
stmt(self, node):
_
_
if isinstance(node, DeclNode):
self.
check
decl(node)
_
_
elif isinstance(node, AssignNode):
self.
check
assign(node)
_
_
elif isinstance(node, WhileNode):
self.
check
while(node)
_
_
check
decl(self, node):
_
_
if node.name in self.table:
self.problems.append(
f"[SEMANTIK] Baris {node.row}: variabel '{node.name}' sudah "
f"pernah dideklarasikan sebelumnya.
"
)
else:
self.table[node.name] = node.dtype
check
assign(self, node):
_
_
if node.name not in self.table:
self.problems.append(
f"[SEMANTIK] Baris {node.row}: variabel '{node.name}' belum "
f"dideklarasikan.
"
)
return
def
def
rhs
type = self.
infer(node.value)
_
_
lhs
type = self.table[node.name]
_
if lhs
type == "int" and rhs
type == "float":
_
_
self.problems.append(
f"[SEMANTIK] Baris {node.row}: nilai bertipe 'float' tidak "
f"dapat disimpan ke variabel '{node.name}' bertipe 'int'
"
.
)
check
while(self, node):
_
_
self.
infer(node.condition.left)
_
self.
infer(node.condition.right)
_
for s in node.body:
self.
check
stmt(s)
_
_
infer(self, node):
_
if isinstance(node, NumberNode):
return "float" if isinstance(node.value, float) else "int"
if isinstance(node, NameNode):
if node.name not in self.table:
self.problems.append(
f"[SEMANTIK] Baris {node.row}: variabel '{node.name}' belum "
f"dideklarasikan.
"
)
return "int"
return self.table[node.name]
if isinstance(node, BinaryNode):
lt = self.
infer(node.left)
_
rt = self.
infer(node.right)
_
return "float" if "float" in (lt, rt) else "int"
raise SemanticIssue(f"Node ekspresi tidak dikenal: {node}")
# =====================================================================
# TAHAP 4: GENERASI KODE ANTARA (THREE-ADDRESS CODE)
# =====================================================================
# Skema penerjemahan while-loop yang dipakai (pola label ganda):
#
# L
start:
_
# t = <hitung kondisi>
# if
false t goto L
end
_
_
# <kode body>
# goto L
start
_
# L
end:
_
# =====================================================================
class TACBuilder:
def
init
(self):
__
__
self.lines = []
self.temp_
seq = 0
self.label
seq = 0
_
def
new
temp(self):
_
_
self.temp_
seq += 1
return f"t{self.temp_
seq}"
def
new
label(self):
_
_
self.label
seq += 1
_
return f"L{self.label
seq}"
_
def
emit(self, line):
_
self.lines.append(line)
def build(self, program):
for stmt in program.body:
self.
stmt(stmt)
_
return self.lines
def
stmt(self, node):
_
if isinstance(node, DeclNode):
self.
emit(f"; deklarasi {node.dtype} {node.name}")
_
elif isinstance(node, AssignNode):
place = self.
expr(node.value)
_
self.
emit(f"{node.name} = {place}")
_
elif isinstance(node, WhileNode):
def
def
def
self.
while(node)
_
while(self, node):
_
label
start = self.
_
label
end = self.
_
new
new
label()
_
label()
_
_
_
self.
emit(f"{label
start}:")
_
_
cond
_place = self.
cond(node.condition)
_
self.
emit(f"if
false {cond
_
_
_place} goto {label
_
for s in node.body:
self.
stmt(s)
_
self.
self.
emit(f"goto {label
start}")
_
_
emit(f"{label
end}:")
end}")
_
_
cond(self, node):
_
left = self.
expr(node.left)
_
right = self.
expr(node.right)
_
temp = self.
new
temp()
_
_
self.
emit(f"{temp} = {left} {node.op} {right}")
_
return temp
expr(self, node):
_
if isinstance(node, NumberNode):
return str(node.value)
if isinstance(node, NameNode):
return node.name
if isinstance(node, BinaryNode):
left = self.
expr(node.left)
_
right = self.
expr(node.right)
_
temp = self.
new
temp()
_
_
self.
emit(f"{temp} = {left} {node.op} {right}")
_
return temp
# =====================================================================
# DRIVER: menjalankan seluruh pipeline kompilasi end-to-end
# =====================================================================
def run
_pipeline(source, label="PROGRAM"):
print("
=
" * 70)
print(f" SOURCE CODE - {label}")
print("
=
" * 70)
print(source.strip())
print("\n" + "
=
" * 70)
print(" TAHAP 1: ANALISIS LEKSIKAL")
print("
=
" * 70)
tokens = scan(source)
for t in tokens:
if t.kind != "EOF":
print(t)
print("\n" + "
=
" * 70)
print(" TAHAP 2: ANALISIS SINTAKSIS (AST)")
print("
=
" * 70)
tree = Parser(tokens).parse
_program()
render
ast(tree)
_
print("\n" + "
=
" * 70)
print(" TAHAP 3: ANALISIS SEMANTIK")
print("
=
" * 70)
checker = SemanticChecker()
try:
table = checker.check(tree)
print("Status: LULUS. Tidak ditemukan kesalahan semantik.
print("Tabel Simbol:")
for name, dtype in table.items():
print(f" {name} : {dtype}")
except SemanticIssue as err:
print("Status: GAGAL. Ditemukan kesalahan semantik:")
print(err)
return
print("\n" + "
=
" * 70)
")
if
print(" TAHAP 4: GENERASI KODE ANTARA (THREE-ADDRESS CODE)")
print("
=
" * 70)
tac = TACBuilder().build(tree)
for line in tac:
print(line)
print()
name
== "
main
":
__
__
__
__
# -----------------------------------------------------------------
# CONTOH 1: Program valid - menjumlahkan angka 0..4 dengan while
# -----------------------------------------------------------------
ok = """
program
_
int i;
int sum;
i = 0;
sum = 0;
while (i < 5) {
sum = sum + i;
i = i + 1;
}
"""
run
_pipeline(program
_
ok, label="CONTOH VALID")
# -----------------------------------------------------------------
# CONTOH 2: Program dengan error semantik
# - variabel 'limit' dipakai tanpa dideklarasikan
# - float disimpan ke variabel bertipe int
# -----------------------------------------------------------------
program
bad = """
_
int i;
float rate;
i = 0;
rate = 1.5;
while (i < limit) {
i = rate;
}
"""
run
_pipeline(program
bad, label="CONTOH ERROR SEMANTIK")
