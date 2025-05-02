" Reconhecimento-de-Som "
#include <iostream>
#include <vector>
#include <cmath>
#include <portaudio.h>
#include <algorithm>

#define SAMPLE_RATE 44100
#define FRAMES_PER_BUFFER 1024
#define NUM_SECONDS 3

std::vector<float> buffer;

static int audioCallback(const void *inputBuffer, void *,
                         unsigned long framesPerBuffer,
                         const PaStreamCallbackTimeInfo *,
                         PaStreamCallbackFlags,
                         void *userData) {
    const float *in = static_cast<const float *>(inputBuffer);
    std::vector<float> *data = static_cast<std::vector<float> *>(userData);

    for (unsigned int i = 0; i < framesPerBuffer; ++i) {
        data->push_back(in[i]);
    }

    return paContinue;
}

float calculateEnergy(const std::vector<float> &samples) {
    float energy = 0.0f;
    for (float sample : samples) {
        energy += sample * sample;
    }
    return energy / samples.size();
}

float detectDominantFrequency(const std::vector<float> &samples) {
    // Método simples: encontrar número de zero crossings
    int zeroCrossings = 0;
    for (size_t i = 1; i < samples.size(); ++i) {
        if ((samples[i - 1] > 0 && samples[i] < 0) || (samples[i - 1] < 0 && samples[i] > 0)) {
            zeroCrossings++;
        }
    }
    float freq = (zeroCrossings * SAMPLE_RATE) / (2.0f * samples.size());
    return freq;
}

void classificarSom(float energia, float frequencia) {
    if (energia > 0.05f) {
        if (frequencia > 1500.0f) {
            std::cout << "Assobio detectado!" << std::endl;
        } else {
            std::cout << "Palma detectada!" << std::endl;
        }
    } else if (energia > 0.01f && frequencia > 80 && frequencia < 400) {
        std::cout << "Voz detectada!" << std::endl;
    } else {
        std::cout << "Nenhum som reconhecido." << std::endl;
    }
}

int main() {
    Pa_Initialize();

    PaStream *stream;
    Pa_OpenDefaultStream(&stream, 1, 0, paFloat32, SAMPLE_RATE,
                         FRAMES_PER_BUFFER, audioCallback, &buffer);

    Pa_StartStream(stream);
    std::cout << "Gravando por " << NUM_SECONDS << " segundos..." << std::endl;
    Pa_Sleep(NUM_SECONDS * 1000);
    Pa_StopStream(stream);
    Pa_CloseStream(stream);
    Pa_Terminate();

    float energia = calculateEnergy(buffer);
    float frequencia = detectDominantFrequency(buffer);
    std::cout << "Energia: " << energia << " | Frequência: " << frequencia << " Hz\n";

    classificarSom(energia, frequencia);

    return 0;
}
