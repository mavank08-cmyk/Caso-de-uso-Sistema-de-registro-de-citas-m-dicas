# Caso-de-uso-Sistema-de-registro-de-citas-m-dicas
use sistema_citas_medicas
use sistema_citas_medicas
db.createCollection("pacientes")
db.createCollection("medicos")
db.createCollection("citas")
show collections
for (let i = 6; i <= 30; i++) {
    db.pacientes.insertOne({
        _id: i,
        nombre: "Paciente " + i,
        edad: 20 + (i % 45),
        genero: i % 2 === 0 ? "Femenino" : "Masculino",
        telefono: "30000000" + i,
        correo: "paciente" + i + "@email.com"
    });
}

let especialidades = [
    "Medicina General",
    "Pediatría",
    "Cardiología",
    "Dermatología",
    "Odontología"
];

for (let i = 6; i <= 10; i++) {
    db.medicos.insertOne({
        _id: i,
        nombre: "Medico " + i,
        especialidad: especialidades[(i - 1) % 5],
        consultorio: "10" + i,
        horario: "08:00 - 16:00"
    });
}

let especialidades = [
    "Medicina General",
    "Pediatría",
    "Cardiología",
    "Dermatología",
    "Odontología"
];

for (let i = 6; i <= 10; i++) {
    db.medicos.insertOne({
        _id: i,
        nombre: "Medico " + i,
        especialidad: especialidades[(i - 1) % 5],
        consultorio: "10" + i,
        horario: "08:00 - 16:00"
    });
}

db.pacientes.insertMany([
    {
        _id: 1,
        nombre: "Laura Martínez",
        edad: 28,
        genero: "Femenino",
        telefono: "3001112233",
        correo: "laura@email.com"
    },
    {
        _id: 2,
        nombre: "Carlos Rodríguez",
        edad: 35,
        genero: "Masculino",
        telefono: "3002223344",
        correo: "carlos@email.com"
    },
    {
        _id: 3,
        nombre: "Ana Gómez",
        edad: 42,
        genero: "Femenino",
        telefono: "3003334455",
        correo: "ana@email.com"
    },
    {
        _id: 4,
        nombre: "Pedro Sánchez",
        edad: 31,
        genero: "Masculino",
        telefono: "3004445566",
        correo: "pedro@email.com"
    },
    {
        _id: 5,
        nombre: "Sofía Ramírez",
        edad: 25,
        genero: "Femenino",
        telefono: "3005556677",
        correo: "sofia@email.com"
    }
])

db.pacientes.countDocuments()
db.medicos.countDocuments()
db.citas.countDocuments()
db.pacientes.find()
db.medicos.find()
db.citas.find()
db.citas.aggregate([
    {
        $lookup: {
            from: "pacientes",
            localField: "pacienteId",
            foreignField: "_id",
            as: "datosPaciente"
        }
    },
    {
        $lookup: {
            from: "medicos",
            localField: "medicoId",
            foreignField: "_id",
            as: "datosMedico"
        }
    },
    {
        $unwind: "$datosPaciente"
    },
    {
        $unwind: "$datosMedico"
    },
    {
        $project: {
            _id: 1,
            paciente: "$datosPaciente.nombre",
            medico: "$datosMedico.nombre",
            especialidad: "$datosMedico.especialidad",
            fechaInicio: 1,
            estado: 1
        }
    }
])
db.citas.aggregate([
    {
        $lookup: {
            from: "citas",
            let: {
                medicoActual: "$medicoId",
                inicioActual: "$fechaInicio",
                finActual: "$fechaFin",
                idActual: "$_id"
            },
            pipeline: [
                {
                    $match: {
                        $expr: {
                            $and: [
                                { $eq: ["$medicoId", "$$medicoActual"] },
                                { $ne: ["$_id", "$$idActual"] },
                                { $lt: ["$fechaInicio", "$$finActual"] },
                                { $gt: ["$fechaFin", "$$inicioActual"] }
                            ]
                        }
                    }
                }
            ],
            as: "crucesHorario"
        }
    },
    {
        $match: {
            "crucesHorario.0": { $exists: true }
        }
    },
    {
        $project: {
            medicoId: 1,
            fechaInicio: 1,
            fechaFin: 1,
            crucesHorario: 1
        }
    }
])
db.citas.aggregate([
    {
        $lookup: {
            from: "medicos",
            localField: "medicoId",
            foreignField: "_id",
            as: "medico"
        }
    },
    {
        $unwind: "$medico"
    },
    {
        $group: {
            _id: "$medico.especialidad",
            totalCitas: { $sum: 1 }
        }
    },
    {
        $sort: {
            totalCitas: -1
        }
    }
])
db.citas.aggregate([
    {
        $group: {
            _id: "$estado",
            cantidad: { $sum: 1 }
        }
    },
    {
        $group: {
            _id: null,
            estados: {
                $push: {
                    estado: "$_id",
                    cantidad: "$cantidad"
                }
            },
            totalCitas: { $sum: "$cantidad" },
            totalCanceladas: {
                $sum: {
                    $cond: [
                        { $eq: ["$_id", "Cancelada"] },
                        "$cantidad",
                        0
                    ]
                }
            }
        }
    },
    {
        $project: {
            _id: 0,
            estados: 1,
            totalCitas: 1,
            totalCanceladas: 1,
            porcentajeCancelacion: {
                $multiply: [
                    { $divide: ["$totalCanceladas", "$totalCitas"] },
                    100
                ]
            }
        }
    }
])
db.pacientes.insertOne({
    _id: 31,
    nombre: "Daniela López",
    edad: 29,
    genero: "Femenino",
    telefono: "3015556677",
    correo: "daniela@email.com"
})
db.pacientes.findOne({ _id: 31 })
db.pacientes.find()
db.pacientes.findOne({ _id: 5 })
db.pacientes.updateOne(
    { _id: 31 },
    {
        $set: {
            telefono: "3028889900"
        }
    }
)
db.pacientes.findOne({ _id: 31 })
db.pacientes.insertOne({
    _id: 999,
    nombre: "Paciente Temporal",
    edad: 30,
    genero: "Masculino",
    telefono: "3009999999",
    correo: "temporal@email.com"
})
db.pacientes.deleteOne({
    _id: 999
})
db.pacientes.findOne({
    _id: 999
})
db.pacientes.find({
    edad: { $gt: 30 }
})
db.pacientes.find({
    edad: {
        $gte: 25,
        $lte: 40
    }
})
db.medicos.find({
    especialidad: "Cardiología"
})
db.citas.find({
    estado: {
        $in: ["Atendida", "Cancelada"]
    }
})
db.citas.aggregate([
    {
        $lookup: {
            from: "pacientes",
            localField: "pacienteId",
            foreignField: "_id",
            as: "paciente"
        }
    },
    {
        $lookup: {
            from: "medicos",
            localField: "medicoId",
            foreignField: "_id",
            as: "medico"
        }
    },
    {
        $unwind: "$paciente"
    },
    {
        $unwind: "$medico"
    },
    {
        $project: {
            _id: 1,
            nombrePaciente: "$paciente.nombre",
            nombreMedico: "$medico.nombre",
            especialidad: "$medico.especialidad",
            fechaInicio: 1,
            estado: 1
        }
    }
])
db.pacientes.aggregate([
    {
        $count: "totalPacientes"
    }
])
db.citas.aggregate([
    {
        $group: {
            _id: "$estado",
            total: { $sum: 1 }
        }
    }
])
db.pacientes.aggregate([
    {
        $group: {
            _id: null,
            promedioEdad: {
                $avg: "$edad"
            }
        }
    },
    {
        $project: {
            _id: 0,
            promedioEdad: 1
        }
    }
])
db.citas.aggregate([
    {
        $group: {
            _id: "$medicoId",
            totalCitas: {
                $sum: 1
            }
        }
    },
    {
        $sort: {
            totalCitas: -1
        }
    }
])
db.citas.aggregate([
    {
        $group: {
            _id: null,
            totalCitas: {
                $sum: 1
            },
            canceladas: {
                $sum: {
                    $cond: [
                        { $eq: ["$estado", "Cancelada"] },
                        1,
                        0
                    ]
                }
            }
        }
    },
    {
        $project: {
            _id: 0,
            totalCitas: 1,
            canceladas: 1,
            porcentajeCancelacion: {
                $multiply: [
                    { $divide: ["$canceladas", "$totalCitas"] },
                    100
                ]
            }
        }
    }
])
db.citas.aggregate([
    {
        $lookup: {
            from: "medicos",
            localField: "medicoId",
            foreignField: "_id",
            as: "medico"
        }
    },
    {
        $unwind: "$medico"
    },
    {
        $group: {
            _id: "$medico.especialidad",
            totalCitas: {
                $sum: 1
            }
        }
    },
    {
        $sort: {
            totalCitas: -1
        }
    }
])

