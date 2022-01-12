'''
SECCIONES PARALELAS A MUROS CON SELECCIÓN DESDE LA INTERFAZ DE USUARIO
Ejercicio desarrollado en clase. ITTI
https://itti.es/official-expert-en-automatizacion-de-procesos-bim/
diegojsanchez@gmail.com
10/01/2022
'''

#BIBLIOTECAS
import clr 

clr.AddReference('RevitAPI') 
from Autodesk.Revit.DB import Document, SpecTypeId, UnitUtils, FilteredElementCollector, ViewFamilyType, BuiltInParameter, Curve, XYZ, Transform, BoundingBoxXYZ, ViewSection

clr.AddReference('RevitAPIUI')
from Autodesk.Revit.UI import Selection

clr.AddReference('RevitNodes')
import Revit
clr.ImportExtensions(Revit.GeometryConversion)

clr.AddReference('RevitServices')
from RevitServices.Persistence import DocumentManager
from RevitServices.Transactions import TransactionManager
doc = DocumentManager.Instance.CurrentDBDocument
uiapp = DocumentManager.Instance.CurrentUIApplication
uidoc = uiapp.ActiveUIDocument


#FUNCIONES
def unidades_modelo_a_internas(x):
    '''Uso: Esta función convierte de unidades internas a metros. Cuidado: Unidades de longitud.'''
    if int(doc.Application.VersionNumber) >= 2022:
        unidadModelo = Document.GetUnits(doc).GetFormatOptions(SpecTypeId.Length).GetUnitTypeId()
        return UnitUtils.ConvertToInternalUnits(x, unidadModelo)
    else:
        unidadModelo = Document.GetUnits(doc).GetFormatOptions(UnitType.UT_Length).DisplayUnits
        return UnitUtils.ConvertToInternalUnits(x, unidadModelo)    

def id_por_tipo_de_familia_de_vista(arg):
    '''Uso: Entrada: Salida:'''
    diccionario = dict()
    for v in FilteredElementCollector(doc).OfClass(ViewFamilyType).ToElements():
        diccionario.Add(str(v.ViewFamily), v.Id) #Add(clave, valor)
    return diccionario[arg] #ElementId
    
def seccion_paralela_por_curva(curva, desfase, altura):
    '''Uso: Crea secciones paralelas a una curva.\nEntrada: curva (Revit o Dynamo); valor desfase (Unidades de modelo); altura (Unidades de modelo).\nSalida: Sección (ViewSection) .'''
    #Conversión de unidades
    o = unidades_modelo_a_internas(desfase)
    h = unidades_modelo_a_internas(float(altura))
    #Revisar tipo de curva
    if str(curva.GetType()) == 'Autodesk.DesignScript.Geometry.Line':
        c = curva.ToRevitType()
    elif str(curva.GetType()) == 'Autodesk.Revit.DB.ModelLine':
        c = curva.Location.Curve
    else:
        c = curva
    
    #Obtener el punto 3D al principio y al final de la curva
    i = c.GetEndPoint(0) #XYZ
    f = c.GetEndPoint(1) #XYZ
    d = f - i #XYZ
    #Longitud
    l = d.GetLength()
    
    #Definir los ejes
    x = d.Normalize() #XYZ
    y = XYZ.BasisZ #XYZ
    z = x.CrossProduct(y) #XYZ
    
    #Crear una instancia de la clase Transform
    t = Transform.Identity
    t.Origin = i + 0.5*d
    t.BasisX = x
    t.BasisY = y
    t.BasisZ = z
    
    #Crear la boundingbox y modificarla
    caja = BoundingBoxXYZ()
    caja.Transform = t
    caja.Min = XYZ(-(0.5*l + o), i.Z - o, - o )
    caja.Max = XYZ(0.5*l + o, h + o, o)
    
    #Crear la sección
    TransactionManager.Instance.EnsureInTransaction(doc)
    seccion = ViewSection.CreateSection(doc, id_por_tipo_de_familia_de_vista('Section'), caja)
    TransactionManager.Instance.TransactionTaskDone()
    
    return seccion

#ENTRADAS
elementos = uidoc.Selection.PickElementsByRectangle('Por favor, seleccionar los muros deseados.') #614
desfase = IN[0]
salida = []

#CÓDIGO
for elemento in elementos:
    if str(elemento.GetType()) == 'Autodesk.Revit.DB.Wall':
        #Información
        altura = elemento.get_Parameter(BuiltInParameter.WALL_USER_HEIGHT_PARAM).AsValueString() #Altura desconectada
        curva = elemento.Location.Curve
        #Crear la sección
        seccion = seccion_paralela_por_curva(curva, desfase,  altura)
        salida.append(seccion)        
    else:
        pass

#SALIDA
OUT = salida
