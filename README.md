# solid-emr-poc

Prueba de concepto de un **Expediente Médico Electrónico** construido sobre **[Solid](https://solidproject.org/)**. Cada paciente y cada médico tiene su propio pod, y el acceso a la información médica se controla mediante archivos **ACL** (Access Control List) de Solid.

Este repositorio contiene los **datos** de un servidor Solid, compatibles con un [Community Solid Server (CSS)](https://github.com/CommunitySolidServer/CommunitySolidServer), con pacientes, médicos y expedientes clínicos de ejemplo, pensados para demostrar el modelo de permisos.

> Todos los datos son ficticios y las URIs usan `http://localhost:3000`. 

## Estructura del repositorio

```
solid-emr-poc/
└── data/                  
    ├── ana/                   # Pod de la paciente Ana
    ├── carlos/                # Pod del paciente Carlos
    ├── luis/                  # Pod del paciente Luis
    ├── sofia/                 # Pod de la paciente Sofía
    ├── dr-salinas/            # Pod del Dr. Salinas
    ├── dr-quiroga/            # Pod del Dr. Quiroga
    ├── dra-llorente/          # Pod de la Dra. Llorente
    ├── dr-filippi/            # Pod del Dr. Filippi
    └── system/                # Pod de sistema (cuenta administradora)
```

Cada pod de **paciente** sigue este patrón:

```
<paciente>/
├── .acl                       # Permisos raíz del pod 
├── .meta                      # Metadatos del contenedor 
├── README$.markdown           # Página de bienvenida del pod (generada por CSS)
├── README.acl                 # Permisos de la página de bienvenida
├── profile/
│   ├── card$.ttl               # Perfil WebID (foaf:Person) del paciente
│   └── card.acl                # Permisos del perfil
└── medical/
    ├── record.tll$.ttl         # Expediente clínico (RDF Turtle)
    └── record.tll.acl / .acl   # Permisos del expediente médico
```

Cada pod de **médico** solo contiene `profile/` (su WebID) y sus propios `.acl`.

El pod `system/` incluye `system/groups/doctors.ttl`, que define el **grupo `Medicos del Sistema`** y enumera los WebIDs de los médicos que pertenecen a él. Este grupo es el que se referencia desde los `.acl` de los pacientes para dar acceso conjunto a "todos los médicos".

## Modelo de datos del expediente clínico

Cada expediente (`medical/record.tll$.ttl`) es un grafo RDF con un vocabulario de ejemplo y describe:

- **Datos del paciente**: `patientID`, `fullName`, `birthDate`, `gender`, `nationality`, `email`, `phone`, `address`, `emergencyContact`.
- **Diagnósticos** (`Diagnosis`): código ICD, descripción, fecha, médico que diagnosticó y severidad.
- **Medicamentos** (`Medication`): nombre, dosis, frecuencia, fecha y médico que prescribió.
- **Citas** (`Appointment`): fecha, médico tratante, especialidad y notas clínicas.

Ejemplo:

```turtle
<http://localhost:3000/ana/medical/record>
  a <https://example.org/medical#MedicalRecord> ;
  ns0:patientID "PAC-2024-001" ;
  ns0:fullName "Ana Garcia Lopez" ;
  ns0:diagnosis <http://localhost:3000/ana/medical/diag/001> ;
  ns0:medication <http://localhost:3000/ana/medical/med/001> ;
  ns0:appointment <http://localhost:3000/ana/medical/apt/001> .
```

## Control de acceso (ACL)

Los permisos se expresan con el vocabulario `http://www.w3.org/ns/auth/acl#` en archivos `.acl` junto a cada recurso. El patrón sobre un expediente médico es:

| Autorización | Agente | Acceso |
|---|---|---|
| `#patient` | El propio paciente (su WebID) | Lectura, escritura, control |
| `#doctors` | Grupo `doctors.ttl#group` (todos los médicos del sistema) | Lectura |
| `#admin` | WebID del sistema (`system/profile/card#me`) | Lectura, escritura, control |
| `#public` | Cualquier agente (`foaf:Agent`) | Lectura (solo en algunos pods) |

> **Nota:** puede que en algunos expedientes la autorización `#public` otorge **lectura a `foaf:Agent`** sobre el contenido de expediente, en dado caso se modificó al momento de estar experimentando con los pods.

Los pods individuales siguen el patrón por defecto de Community Solid Server: la página de inicio del pod es pública, y el resto del pod es privado salvo reglas explícitas como las anteriores.

## Personas de ejemplo

| Pod | Rol |
|---|---|
| `ana`, `carlos`, `luis`, `sofia` | Pacientes, cada uno con su expediente clínico |
| `dr-salinas`, `dr-quiroga`, `dra-llorente`, `dr-filippi` | Médicos, miembros del grupo `Medicos del Sistema` |
| `system` | Cuenta/pod administrador del servidor |

> Repositorio de exploración, sin aplicación más allá de los datos.
