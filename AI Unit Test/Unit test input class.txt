package com.example.demo.services;

import com.example.demo.entities.Brand;
import com.example.demo.entities.Model;
import com.example.demo.repositories.BrandRepository;
import com.example.demo.repositories.ModelRepository;
import com.example.demo.mappers.ModelMapper;
import com.example.demo.dtos.requests.CreateModelRequest;
import com.example.demo.dtos.requests.UpdateModelRequest;
import com.example.demo.dtos.responses.GetModelDetailsResponse;
import com.example.demo.dtos.responses.GetModelResponse;
import com.example.demo.constants.CacheConstants;
import com.example.demo.constants.UIMessages;
import com.example.demo.utils.result.*;
import lombok.RequiredArgsConstructor;
import org.springframework.cache.annotation.CacheEvict;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Isolation;
import org.springframework.transaction.annotation.Propagation;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;

@Service
@RequiredArgsConstructor
@Transactional(isolation = Isolation.READ_COMMITTED, propagation = Propagation.REQUIRED)
public class ModelServiceImpl implements ModelService {
    private final ModelRepository modelRepository;
    private final BrandRepository brandRepository;
    private final ModelMapper modelMapper;

    @Cacheable(value = CacheConstants.MODELS, key = "#id")
    @Override
    public DataResult<GetModelDetailsResponse> getById(Long id) {
        Model model = modelRepository.findById(id).orElse(null);
        if (model == null) {
            return new ErrorDataResult<>(null, UIMessages.NOT_FOUND_DATA);
        }

        GetModelDetailsResponse response = modelMapper.toDetailsDto(model);

        return new SuccessDataResult<>(response, UIMessages.SUCCESS);
    }

    @Cacheable(value = CacheConstants.MODELS, key = CacheConstants.ALL_KEY)
    @Override
    public DataResult<List<GetModelResponse>> getAll() {
        List<GetModelResponse> response;
        List<Model> models = modelRepository.findAll();

        response = modelMapper.toDtoList(models);

        return new SuccessDataResult<>(response, UIMessages.SUCCESS);
    }

    @CacheEvict(value = {CacheConstants.BRANDS, CacheConstants.MODELS}, allEntries = true)
    @Override
    public Result add(CreateModelRequest createModelRequest) {
        Brand brand = brandRepository.findById(createModelRequest.getBrandId()).orElse(null);
        if (brand == null) {
            return new ErrorResult(UIMessages.ERROR);
        }

        Model model = modelMapper.toEntity(createModelRequest);
        model.setBrand(brand);
        modelRepository.save(model);

        return new SuccessResult(UIMessages.SUCCESS);
    }

    @CacheEvict(value = {CacheConstants.BRANDS, CacheConstants.MODELS}, allEntries = true)
    @Override
    public Result update(UpdateModelRequest updateModelRequest) {
        Model model = modelRepository.findById(updateModelRequest.getId()).orElse(null);
        if (model == null) {
            return new ErrorResult(UIMessages.NOT_FOUND_DATA);
        }

        Brand brand = brandRepository.findById(updateModelRequest.getBrandId()).orElse(null);
        if (brand == null) {
            return new ErrorResult(UIMessages.ERROR);
        }

        modelMapper.toEntity(updateModelRequest, model);
        model.setBrand(brand);
        modelRepository.save(model);

        return new SuccessResult(UIMessages.SUCCESS);
    }

    @CacheEvict(value = {CacheConstants.BRANDS, CacheConstants.MODELS}, allEntries = true)
    @Override
    public Result delete(Long id) {
        Model model = modelRepository.findById(id).orElse(null);
        if (model == null) {
            return new ErrorResult(UIMessages.NOT_FOUND_DATA);
        }

        modelRepository.delete(model);

        return new SuccessResult(UIMessages.SUCCESS);
    }
}
