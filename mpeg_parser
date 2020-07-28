#include "tsCommon.h"
#include "tsTransportStream.h"
#include "stdio.h"

int main(int argc, char* argv[], char* envp[])
{
    // TODO - open file
    FILE* pFile;
    pFile = fopen("example_new.ts", "rb");
    FILE* Audio;
    Audio = fopen("PID_136.mp2", "wb");
    // END TODO (raczej git)


    xTS_PacketHeader    TS_PacketHeader;
    xTS_AdaptationField TS_AdaptationField;
    int32_t TS_PacketId = 0;
    uint8_t* TS_PacketBuffer;

    TS_PacketBuffer = (uint8_t*)malloc(sizeof(uint8_t) * 188);
    xPES_Assembler PES_Assembler;
    xPES_PacketHeader PES_PacketHeader;
    PES_Assembler.Init(136);

    while (!feof(pFile))
    {
        // TODO - read from file
         //size_t read_bytes=  fread(&TS_PacketBuffer, sizeof(uint8_t), 1, pFile);
        fread(TS_PacketBuffer, 188, 1, pFile);
        uint8_t* TS_PacketBuffer_2 = TS_PacketBuffer;
        TS_PacketHeader.Reset();
        TS_PacketHeader.Parse(TS_PacketBuffer);

        if (TS_PacketHeader.getPID() == 136)
        {
            printf("%010d ", TS_PacketId);
            TS_PacketHeader.Print();

            if (TS_PacketHeader.hasAdaptationField()) {
                /* TODO PARSE AF */
                TS_AdaptationField.Reset();
                TS_AdaptationField.Parse(TS_PacketBuffer, TS_PacketHeader.getAF());
                TS_AdaptationField.Print();
            }
            //PES_PacketHeader.Parse(TS_PacketBuffer);
            //PES_PacketHeader.Print();
        }

        /*HERE*/
        if (TS_PacketHeader.getPID() == 136 && TS_PacketHeader.getS() == 1)
        {
            printf(" Started ");
            PES_PacketHeader.Parse(TS_PacketBuffer);
            PES_PacketHeader.Print();
            fwrite(TS_PacketBuffer + PES_PacketHeader.Parse(TS_PacketBuffer) + 6 + 3 + PES_PacketHeader.get_m_header_data_len(), 188 - (PES_PacketHeader.Parse(TS_PacketBuffer) + 6 + 3 + PES_PacketHeader.get_m_header_data_len()), 1, Audio);
        }


        if (TS_PacketHeader.getPID() == 136 && TS_PacketHeader.getCC() == 15)
        {
            printf(" Finished");
        }

        if (TS_PacketHeader.getPID() == 136 && TS_PacketHeader.getS() == 0)
        {
            fwrite(TS_PacketBuffer + 4, 184, 1, Audio);
        }

        xPES_Assembler::eResult Result = PES_Assembler.AbsorbPacket(TS_PacketBuffer, &TS_PacketHeader, &TS_AdaptationField);
        switch (Result)
        {
        case xPES_Assembler::eResult::StreamPackedLost: printf("PcktLost"); break;
        case xPES_Assembler::eResult::AssemblingStarted: printf("Started"); PES_Assembler.PrintPESH(); break;
        case xPES_Assembler::eResult::AssemblingContinue: printf("Continue"); break;
        case xPES_Assembler::eResult::AssemblingFinished: printf("Finished"); printf("PES: Len=%d", PES_Assembler.getNumPacketBytes()); break;
        default: break;
        }
        printf("\n");

        TS_PacketId++;

    }
    fclose(pFile);
    fclose(Audio);
}

#pragma once
#include "tsCommon.h"
#include <string>

/*
MPEG-TS packet:
`        3                   2                   1                   0  `
`      1 0 9 8 7 6 5 4 3 2 1 0 9 8 7 6 5 4 3 2 1 0 9 8 7 6 5 4 3 2 1 0  `
`     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ `
`   0 |                             Header                            | `
`     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ `
`   4 |                  Adaptation field + Payload                   | `
`     |                                                               | `
` 184 |                                                               | `
`     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ `


MPEG-TS packet header:
`        3                   2                   1                   0  `
`      1 0 9 8 7 6 5 4 3 2 1 0 9 8 7 6 5 4 3 2 1 0 9 8 7 6 5 4 3 2 1 0  `
`     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ `
`   0 |       SB      |E|S|T|           PID           |TSC|AFC|   CC  | `
`     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ `

Sync byte                    (SB ) :  8 bits
Transport error indicator    (E  ) :  1 bit
Payload unit start indicator (S  ) :  1 bit
Transport priority           (T  ) :  1 bit
Packet Identifier            (PID) : 13 bits
Transport scrambling control (TSC) :  2 bits
Adaptation field control     (AFC) :  2 bits
Continuity counter           (CC ) :  4 bits
*/


//=============================================================================================================================================================================

class xTS
{
public:
    static constexpr uint32_t TS_PacketLength = 188;
    static constexpr uint32_t TS_HeaderLength = 4;

    static constexpr uint32_t PES_HeaderLength = 6;

    static constexpr uint32_t BaseClockFrequency_Hz = 90000; //Hz
    static constexpr uint32_t ExtendedClockFrequency_Hz = 27000000; //Hz
    static constexpr uint32_t BaseClockFrequency_kHz = 90; //kHz
    static constexpr uint32_t ExtendedClockFrequency_kHz = 27000; //kHz
    static constexpr uint32_t BaseToExtendedClockMultiplier = 300;
};

//=============================================================================================================================================================================

class xTS_PacketHeader
{
public:
    enum class ePID : uint16_t
    {
        PAT = 0x0000,
        CAT = 0x0001,
        TSDT = 0x0002,
        IPMT = 0x0003,
        NIT = 0x0010, //DVB specific PID
        SDT = 0x0011, //DVB specific PID
        NuLL = 0x1FFF,
    };

protected:
    //TODO - header fields
    uint16_t SB;
    uint16_t E;
    uint16_t S;
    uint16_t T;
    uint16_t PID;
    uint16_t TSC;
    uint16_t AF;
    uint16_t CC;

public:
    void     Reset();
    int32_t  Parse(const uint8_t* Input);
    void     Print()  const;

public:
    //TODO - direct acces to header values
    int getAF();
    int getPID();
    int getSB();
    int getS();
    int getCC();
public:
    //TODO
    uint8_t AFC;
    bool     hasAdaptationField() const { return (AF == 2 || AF == 3) ? true : false; }
    bool     hasPayload() const { }
};

//=============================================================================================================================================================================
class xTS_AdaptationField
{
protected:
    uint16_t AFL;
    uint16_t DC;
    uint16_t RA;
    uint16_t SP;
    uint16_t PR;
    uint16_t OR;
    uint16_t SPf;
    uint16_t TP;
    uint16_t EX;
    uint64_t PCR = 0;

public:
    void Reset();
    int32_t Parse(const uint8_t* Input, uint8_t AdaptationFieldControl);
    void Print() const;
public:
    //derrived values
    uint32_t getNumBytes() const { }
};
//=============================================================================================================================================================================

class xPES_PacketHeader
{
public:
    enum eStreamId : uint8_t
    {
        eStreamId_program_stream_map = 0xBC,
        eStreamId_padding_stream = 0xBE,
        eStreamId_private_stream_2 = 0xBF,
        eStreamId_ECM = 0xF0,
        eStreamId_EMM = 0xF1,
        eStreamId_program_stream_directory = 0xFF,
        eStreamId_DSMCC_stream = 0xF2,
        eStreamId_ITUT_H222_1_type_E = 0xF8,
    };

protected:
    //PES packet header
    uint32_t m_PacketStartCodePrefix;   //24
    uint8_t  m_StreamId;                //8
    uint16_t m_PacketLength;            //16
    uint8_t AFL;
    uint8_t m_header_data_len; //8

public:
    void     Reset();
    int32_t  Parse(const uint8_t* Input);
    void     Print() const;

public:
    //PES packet header
    uint32_t getPacketStartCodePrefix() const { return m_PacketStartCodePrefix; }
    uint8_t  getStreamId() const { return m_StreamId; }
    uint16_t getPacketLength() const { return m_PacketLength; }
    uint8_t get_m_header_data_len() const { return m_header_data_len; }
};
//=============================================================================================================================================================================

class xPES_Assembler
{
public:
    enum class eResult : int32_t
    {
        UnexpectedPID = 1,
        StreamPackedLost,
        AssemblingStarted,
        AssemblingContinue,
        AssemblingFinished,
    };

protected:
    //setup
    int32_t  m_PID;
    //buffer
    uint8_t* m_Buffer;
    uint32_t m_BufferSize;
    uint32_t m_DataOffset;
    //operation
    int8_t            m_LastContinuityCounter;
    bool              m_Started;
    xPES_PacketHeader m_PESH;

public:
    xPES_Assembler();
    ~xPES_Assembler();

    void    Init(int32_t PID);

    eResult AbsorbPacket(const uint8_t* TransportStreamPacket, const xTS_PacketHeader* PacketHeader, const xTS_AdaptationField* AdaptationField)
    {
        return eResult();
    }

    void     PrintPESH() const { m_PESH.Print(); }
    uint8_t* getPacket() { return m_Buffer; }
    int32_t  getNumPacketBytes() const { return m_DataOffset; }

protected:
    void xBufferReset();
    void xBufferAppend(const uint8_t* Data, int32_t Size);

};

#include "tsTransportStream.h"
#include <iostream>
#include <iomanip>

//=============================================================================================================================================================================
// xTS_PacketHeader
//=============================================================================================================================================================================

void xTS_PacketHeader::Reset()
{
}

int32_t xTS_PacketHeader::Parse(const uint8_t* Input)
{
    SB = Input[0];
    E = Input[1] & 128 ? 1 : 0;
    S = Input[1] & 64 ? 1 : 0;
    T = Input[1] & 32 ? 1 : 0;
    PID = ((Input[1] & 31) << 8) | Input[2];
    TSC = (Input[3] & 192) >> 6;
    AF = (Input[3] & 48) >> 4;
    CC = Input[3] & 15;

    return int32_t();
}

void xTS_PacketHeader::Print() const
{
    std::cout << "TS: SB=" << SB << " E=" << E << " S=" << S << " T=" << T << " PID=" << PID << " TSC=" << TSC << " AF=" << AF << " CC=" << std::setw(2) << CC << " ";
}

int xTS_PacketHeader::getAF()
{
    return AF;
}

int xTS_PacketHeader::getPID()
{
    return PID;
}
int xTS_PacketHeader::getSB()
{
    return SB;
}
int xTS_PacketHeader::getS()
{
    return S;
}
int xTS_PacketHeader::getCC()
{
    return CC;
}

//=============================================================================================================================================================================
// xTS_AdaptationField
//=============================================================================================================================================================================


void xTS_AdaptationField::Reset()
{

}

int32_t xTS_AdaptationField::Parse(const uint8_t* Input, uint8_t AdaptationFieldControl)
{
    AFL = Input[4];

    if (AFL > 0) {
        DC = (Input[5] & 128) >> 7;
        RA = (Input[5] & 64) >> 6;
        SP = (Input[5] & 32) >> 5;
        PR = (Input[5] & 16) >> 4;
        OR = (Input[5] & 8) >> 3;
        SPf = (Input[5] & 4) >> 2;
        TP = (Input[5] & 2) >> 1;
        EX = (Input[5] & 1);
        /*
        //======== DODATKOWE =========
        if (PR == 1) {
            uint64_t PCR_base = (Input[6] << 25) | (Input[7] << 17) | (Input[8] << 9) | (Input[9] << 1) | ((Input[10] & 128) >> 7);
            uint64_t PCR_ext = ((Input[10] & 1) << 8) | Input[11];
            PCR = PCR_base * 300 + PCR_ext;
        }
        //============================
        */
    }

    if (AdaptationFieldControl == 2) {
        //AF - 188 B
    }
    else if (AdaptationFieldControl == 3) {
        //AF and Payload 1+L
    }

    return int32_t();
}

void xTS_AdaptationField::Print() const
{
    std::cout << "AF: " << "L=" << AFL << " DC=" << DC << " RA=" << RA << " SP=" << SP << " PR=" << PR << " OR=" << OR << " SP=" << SPf << " TP=" << TP << " EX=" << EX;
    if (PR == 1) {
        std::cout << " PCR=" << PCR;
    }
}


//=============================================================================================================================================================================

void xPES_PacketHeader::Reset()
{
}

int32_t xPES_PacketHeader::Parse(const uint8_t* Input)
{
    AFL = Input[4];
    //std::cout << int(Input[0])<<" " << int(Input[1]) << " " << int(Input[2]) << " " << int(Input[3]) << " " << int(Input[4]) << " " << int(Input[5]) << " " << int(Input[6]) << " " << int(Input[7]) << " " << int(Input[16]) << " " << " " << int(Input[17]) << " " << int(Input[18]) << " " << int(Input[19]);
    m_PacketStartCodePrefix = (Input[4 + 1 + AFL] << 16) | (Input[5 + 1 + AFL] << 8) | Input[6 + 1 + AFL];   //24
    m_StreamId = Input[7 + 1 + AFL];				//8
    m_PacketLength = Input[8 + 1 + AFL] << 8 | Input[9 + 1 + AFL];            //16
    //std::cout << int(Input[12] | Input[13] | Input[14]);
    m_header_data_len = Input[12 + 1 + AFL];
    return AFL;
}

void xPES_PacketHeader::Print() const
{
    std::cout << " PES: " << int(m_PacketStartCodePrefix) << " ";
    std::cout << "Stream id:  " << int(m_StreamId) << " ";
    std::cout << "PES Packet length: " << int(m_PacketLength);
    std::cout << "\n";
}

xPES_Assembler::xPES_Assembler()
{
}

xPES_Assembler::~xPES_Assembler()
{
}

void xPES_Assembler::Init(int32_t PID)
{
}

void xPES_Assembler::xBufferReset()
{
}

void xPES_Assembler::xBufferAppend(const uint8_t* Data, int32_t Size)
{
}
